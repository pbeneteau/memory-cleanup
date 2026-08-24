# memory-cleanup

A Claude skill that audits Claude's persistent memory for stale, outdated, or superseded entries — and proposes them for deletion instead of removing anything on its own.

## Why

Persistent memory in Claude accumulates facts about abandoned projects, dead ideas, and outdated status the same way any long-running system accumulates cruft. Nothing expires on its own, so it just sits there.

## What it does

1. **Detect** — lists all memory files, sorts by how long since each was last touched, and weighs that against the folder it lives in (`/areas/` project files going stale is a much stronger signal than `/profile.md` or `/topics/` files simply not changing, since identity and habits are supposed to stay put).
2. **Propose** — surfaces a short list of candidates with a one-line summary of each, and asks which ones to remove. Never deletes without an explicit, itemized yes.
3. **Execute** — deletes only what was confirmed, then checks the remaining files for dangling references to what just got removed and cleans those up too.

## Install

Drop `SKILL.md` into your skills directory, or upload `memory-cleanup.skill` wherever Claude skills are installed.

## Requires

`memory_list`, `memory_read`, `memory_delete`, `memory_str_replace` (Claude's persistent memory tools). `ask_user_input_v0` if available, for tappable proposals.
