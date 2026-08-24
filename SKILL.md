---
name: memory-cleanup
description: "Détecte les entrées obsolètes, périmées ou inutiles dans la mémoire persistante d'un agent Claude — claude.ai, Claude Code, Codex, ou tout agent avec une couche de mémoire long-terme — et les propose à l'utilisateur pour suppression, sans jamais rien supprimer sans confirmation explicite item par item. À utiliser dès que l'utilisateur demande de nettoyer, auditer, faire le tri dans, ou vérifier sa mémoire, mentionne des infos périmées, stale ou dépassées que l'agent aurait gardées, ou demande ce qui pourrait ne plus être à jour, même formulé de façon informelle (\"check ta mémoire\", \"y'a des trucs à virer ?\", \"fais du ménage\"). Ne PAS utiliser ce skill pour supprimer quoi que ce soit de sa propre initiative — la détection et la proposition sont automatiques, la suppression ne l'est jamais."
compatibility: "N'importe quel agent Claude (ou compatible) ayant accès en lecture/écriture à sa propre mémoire persistante — claude.ai (tools memory_*), Claude Code (CLAUDE.md + auto memory), Codex (AGENTS.md + mémoire persistante), ou une couche mémoire tierce installée. Ne code aucun chemin ni nom de tool en dur : s'appuie sur ce que l'environnement d'exécution expose réellement au moment où il tourne."
---

# Memory Cleanup

Détecte les entrées obsolètes, périmées ou inutiles dans la mémoire persistante d'un agent — et les propose à l'utilisateur pour suppression. Ne supprime jamais rien de sa propre initiative.

## Règle absolue

**Ne jamais supprimer ou modifier une entrée de mémoire sans confirmation explicite de l'utilisateur pour cet item précis.** La détection et la proposition sont automatiques dès que ce skill se déclenche. La suppression ne l'est jamais — même après un feu vert général en début d'échange : ça vaut confirmation du principe, pas des items précis tant qu'ils n'ont pas été listés un par un.

## Où vit la mémoire ici ?

Identifie d'abord ton environnement — tu sais déjà comment lister, lire et éditer ta propre mémoire dans cet environnement précis, et cette mécanique change plus vite que ce skill ne pourrait la documenter correctement :

- Claude.ai / app Claude → tools memory_list / memory_read / memory_delete / memory_str_replace.
- Claude Code → CLAUDE.md et l'auto memory (`/memory` pour naviguer, tools fichier standards pour éditer).
- Codex → AGENTS.md et son répertoire de mémoire persistante.
- Tout autre agent avec une couche mémoire installée (MCP tiers, etc.) → ses tools propres de liste/lecture/suppression.
- Aucune mémoire persistante native détectée dans cet environnement (ex. Cursor sans couche tierce) → dis-le à l'utilisateur, ne fabrique rien à nettoyer.

Ne code jamais en dur un chemin de fichier ou un nom de commande précis dans ce skill — utilise ce que ton environnement expose réellement au moment où tu tournes.

## Phase 1 — Détection

1. Liste l'ensemble de ta mémoire persistante et l'ancienneté de chaque entrée (depuis quand elle n'a pas bougé).
2. Trie par ancienneté décroissante. Un candidat n'en est un que s'il est nettement plus ancien que le reste du corpus — pas de seuil fixe universel : compare à la distribution réelle de cette mémoire précise, pas à un chiffre absolu.
3. L'ancienneté seule est un signal faible, pas une preuve. Avant de proposer quoi que ce soit :
   - Identité, préférences stables, habitudes bougent rarement par nature — être vieux n'y est PAS un signal de péremption.
   - Un projet ou une tâche en cours est le type d'entrée où l'ancienneté est le signal le plus fiable, surtout si le contenu décrit un état "en cours" ou "à faire" qui semble figé depuis longtemps.
   - Lis le contenu complet des candidats potentiels avant de les proposer — pas juste un résumé — pour juger si le sujet semble abandonné, remplacé par autre chose, ou explicitement clos, pas juste vieux.
   - Croise avec la conversation en cours : un pivot, un abandon ou un changement mentionné récemment renforce nettement le signal.
4. Construis une courte liste de candidats — 4 à 6 maximum par lot pour rester lisible — avec pour chacun : de quoi il s'agit en une ligne, et depuis quand il n'a pas bougé.

## Phase 2 — Proposition

5. Présente les candidats, jamais ne les supprime directement. Adapte-toi au canal disponible : options tappables si l'environnement le permet, sinon une liste numérotée avec une question explicite ("Lesquels veux-tu que je supprime ?").
6. Plus de candidats que ce que le canal affiche proprement → répartis en plusieurs lots plutôt que de tronquer silencieusement.
7. Une réponse vague ("nettoie tout", "vas-y") sur une liste qui n'a pas encore été présentée précisément n'est pas une confirmation valable pour la suppression — présente d'abord la liste exacte, confirme ensuite item par item.

## Phase 3 — Exécution

8. Ne supprime ou ne modifie que ce qui a été confirmé, avec les moyens d'édition natifs de ton environnement.
9. Vérifie les références croisées : d'autres entrées peuvent mentionner ou lier ce qui vient d'être supprimé, souvent dans une note fourre-tout type "activité récente". Retire les lignes qui ne parlent plus QUE du sujet supprimé — pas celles qui le mentionnent incidemment dans un contexte encore pertinent par ailleurs.
10. Confirme ce qui a été supprimé en une réponse courte. Pas de récap superflu — juste ce qui a changé.

## Exemple de déroulé complet

- Liste de la mémoire → une quinzaine d'entrées, dont six nettement plus anciennes que le reste du corpus pendant qu'il tourne activement.
- Lecture complète de ces six → certaines décrivent des projets concrets à l'arrêt apparent, une autre une personne liée à une évaluation déjà tranchée. Candidats retenus.
- Proposition à l'utilisateur, jamais de suppression directe.
- Confirmation d'une liste précise → suppression de chaque item confirmé.
- Vérification croisée : une note fourre-tout mentionnait deux des sujets supprimés en passant. Ces lignes précises sont retirées, le reste de la note reste intact.
- Confirmation finale en une réponse courte : quoi a été supprimé, quoi a été vérifié et laissé tel quel.
