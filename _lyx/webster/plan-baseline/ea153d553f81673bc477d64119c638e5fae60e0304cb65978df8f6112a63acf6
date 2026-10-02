# Card 9 — board-skill

**Create:**
- `plugins/ly/skills/board/SKILL.md`

**Prosa:**
- `plugins/ly/skills/INDEX.md`

**Uses:**
- `plugins/ly/skills/ly-drive/SKILL.md`

**Intent:**
Give every agent the `ly:board` skill, so the board is reached through `lyx board` and never by reading or editing `_board/*.json` (Decisions 21, 23).

`SKILL.md` has the same frontmatter shape as `ly-drive`'s (`name: board`, a one-line `description` that triggers on reading or writing the board, creating or promoting an entry, or searching it) and covers:
- The board is read and written only through `lyx board`; never read, grep or edit `_board/*.json` or the rendered README directly.
- Tier meanings (1 Planned: concretized and claimable; 2 Next Up: planned next, not concretized; 3 Someday: loose idea) and that a new entry without `tier` lands at 3, so claimable work passes `tier: 1`.
- Type meanings (`feature`, `bug`, `chore`, `design` — a decision or design document, not shipped code) and that a recipe name goes in `recipe`.
- Creating (`upsert`), promoting (`promote`, demotion through `upsert`), finding (`find`, with `--text`), listing (`list --text` to scan, `get` for one entry), and pruning done entries.
- The payload of each common verb, with one example each, pointing at `lyx board <verb> --help` for the full key list rather than restating it.
- A `body` renders as ordinary markdown with semantic line breaks: a single newline is a soft break, so structure a body with headings and lists.

The skill's own markdown follows semantic line breaks and calls the tracker "the board", never "Bolt".

`INDEX.md` gains a `board` row in the table, linking `board/SKILL.md` with the skill's description.
