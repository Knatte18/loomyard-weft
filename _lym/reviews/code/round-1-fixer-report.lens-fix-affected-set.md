No findings.

- F2: a repo-wide grep for `UpsertTaskSiblings` (excluding `.git`, `.lym`, `_lym`) returns nothing, so no stale reference remains.
- F3: `plugins/ly/skills/board/SKILL.md` is the only plugin skill that mentions the warning, and now covers both promote and upsert. `docs/overview.md` also contains "warning", which I did not open. I did not check whether it describes the promote warning or the board verbs.
- F1 and F4: neither finding's premise is a set that needs rechecking.

Calls made:
1. grep for `UpsertTaskSiblings` across the repo, to check F2's set for leftover references.
2. grep for "warning" under `plugins/` and `docs`, plus `decision add` in the board `SKILL.md`, to check F3 for other docs that describe the board warning.
