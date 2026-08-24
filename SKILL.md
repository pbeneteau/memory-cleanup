---
name: memory-cleanup
description: "Détecte les entrées obsolètes, périmées ou inutiles dans la mémoire persistante de Claude (fichiers accessibles via memory_list/memory_read/memory_delete/memory_str_replace) et les propose à l'utilisateur pour suppression, sans jamais rien supprimer sans confirmation explicite item par item. À utiliser dès que l'utilisateur demande de nettoyer, auditer, faire le tri dans, ou vérifier sa mémoire, mentionne des infos périmées, stale ou dépassées que Claude aurait gardées, ou demande ce que Claude sait qui pourrait ne plus être à jour, même formulé de façon informelle (\"check ta mémoire\", \"y'a des trucs à virer ?\", \"fais du ménage\"). Ne PAS utiliser ce skill pour supprimer quoi que ce soit de sa propre initiative — la détection et la proposition sont automatiques, la suppression ne l'est jamais."
compatibility: "memory_list, memory_read, memory_delete, memory_str_replace (mémoire persistante de l'app Claude / claude.ai). ask_user_input_v0 recommandé pour la phase de proposition sur mobile/claude.ai, sinon liste texte et question explicite."
---

# Memory Cleanup

Nettoie la mémoire persistante de Claude — pas le CLAUDE.md de Claude Code, ni un fichier mémoire de projet. Spécifiquement les fichiers accessibles via `memory_list` / `memory_read` / `memory_delete` / `memory_str_replace` / `memory_append` dans l'app Claude / claude.ai.

## Règle absolue

**Ne jamais appeler `memory_delete`, ni retirer une ligne via `memory_str_replace`, sans confirmation explicite de l'utilisateur pour cet item précis.** La détection de candidats et leur proposition sont automatiques dès que ce skill se déclenche. La suppression ne l'est jamais — même si l'utilisateur a déjà validé une suppression similaire plus tôt dans la conversation, et même si la demande qui déclenche le skill semble déjà être un feu vert général ("nettoie tout ce qui traîne") : ça vaut confirmation du principe, pas des items précis tant qu'ils n'ont pas été listés.

## Phase 1 — Détection

1. `memory_list()` avec `include_preview: true` : chemin, taille, date de dernière modif et aperçu de chaque fichier.
2. Calculer l'ancienneté de chaque fichier (jours depuis la dernière modif, par rapport à la date du jour — `user_time_v0` si besoin).
3. Trier par ancienneté décroissante. Un fichier n'est un candidat que s'il est nettement plus ancien que le reste du corpus — pas de seuil fixe universel (30/60/90 jours selon les gens) : comparer à la distribution réelle des autres fichiers de cette mémoire, pas à un chiffre absolu.
4. L'ancienneté seule est un signal faible, pas une preuve. Avant de proposer quoi que ce soit :
   - `/profile.md` et la plupart des `/topics/` bougent rarement par nature (identité, goûts, habitudes stables) — être vieux n'y est PAS un signal de péremption. Ne pas les proposer par défaut sur ce seul critère.
   - `/areas/` (projets, tâches en cours) est le dossier où l'ancienneté est le signal le plus fiable, surtout si le contenu décrit un état "en cours" ou "à faire" qui semble figé depuis longtemps.
   - Lire le contenu complet des candidats potentiels (`memory_read`, pas juste l'aperçu) avant de les proposer, pour juger si le sujet semble abandonné, remplacé par autre chose, ou explicitement clos — pas juste vieux.
   - Croiser avec la conversation en cours : un pivot, un abandon ou un changement mentionné récemment renforce nettement le signal.
5. Construire une courte liste de candidats — 4 à 6 maximum par lot pour rester lisible — avec pour chacun : le nom du fichier, un résumé d'une ligne de ce qu'il contient, et depuis quand il n'a pas bougé.

## Phase 2 — Proposition

6. Présenter les candidats, jamais les supprimer directement.
   - Sur claude.ai / app mobile : `ask_user_input_v0` en `multi_select` est l'outil naturel (max 4 options par question, jusqu'à 3 questions) — l'utilisateur coche ce qu'il veut vraiment supprimer.
   - Ailleurs (pas d'accès à cet outil) : une liste numérotée en texte avec une question explicite ("Lesquels de ces fichiers veux-tu que je supprime ?").
7. Plus de 4 candidats → répartir sur plusieurs questions plutôt que tronquer la liste silencieusement.
8. Une réponse vague ("oui nettoie tout", "vas-y") sur une liste qui n'a pas encore été présentée précisément n'est pas une confirmation valable pour la suppression — présenter d'abord la liste exacte, confirmer ensuite item par item.

## Phase 3 — Exécution

9. Pour chaque item confirmé par l'utilisateur :
   - Fichier entier à supprimer → `memory_read` (si la version en main date d'avant cette conversation ou est incertaine) puis `memory_delete(path, if_version)`.
   - Une ligne précise dans un fichier à garder par ailleurs → `memory_str_replace` pour retirer uniquement cette ligne, pas tout le fichier.
10. Vérifier les références croisées : d'autres fichiers peuvent contenir un lien `[[nom-du-sujet-supprimé]]` ou juste le mentionner en passant (souvent dans un fichier fourre-tout type `/topics/recent-work.md`). Relire les fichiers plausibles et retirer les lignes qui ne parlent plus QUE du sujet supprimé — pas celles qui le mentionnent incidemment dans un contexte encore pertinent par ailleurs.
11. Confirmer à l'utilisateur ce qui a été supprimé, fichier par fichier, en une réponse courte. Pas de récap superflu — juste la confirmation de ce qui a changé.

## Exemple de déroulé complet

- `memory_list()` → 15 fichiers, dont 6 dans `/areas/` et `/people/` non modifiés depuis 3 à 5 semaines pendant que le reste du corpus tourne activement.
- Lecture complète de ces 6 → certains sont des projets logiciels concrets à l'arrêt apparent, un autre est une personne (co-fondateur) liée à une évaluation d'idée déjà tranchée. Candidats retenus.
- Proposition via `ask_user_input_v0` (multi_select, 4 options max par question).
- L'utilisateur confirme une liste précise → `memory_delete` sur chacun, en relisant d'abord ceux dont la version en main n'est plus fraîche.
- Vérification croisée : un fichier `/topics/recent-work.md` mentionnait deux des sujets supprimés en passant, via des liens `[[...]]`. Ces deux lignes précises sont retirées avec `memory_str_replace` ; le reste du fichier reste intact.
- Confirmation finale en une réponse courte : quoi a été supprimé, quoi a été vérifié et laissé tel quel.
