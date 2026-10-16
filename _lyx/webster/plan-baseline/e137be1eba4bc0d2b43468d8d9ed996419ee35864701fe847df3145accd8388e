# Card 3 — the orch checks the inbox and forks intake, and ly:board matches

**Prosa:**
- `contracts/stencils/orch/orch-template-role.md`
- `plugins/ly/skills/board/SKILL.md`

**Intent:**
D3, D4 and D5 in the two prose files the intake fork and the orch read, one sentence per line.
No test pins this prose (`PATTERN-test-economy`); review guards it.
Both files name only commands the agents' settings allow (`PATTERN-no-denied-recovery`), and neither asks a session to load a skill (`PATTERN-role-skills-typed`): the fork inherits the orch's loaded `ly:board`.

`orch-template-role.md`, Where a finding goes: its last sentence, which has the orch take every issue onto the board itself, is replaced by the inbox check.
- At every wake, the `landed` notice included, the orch runs `lyx board intake list` and reads only its numbers and titles; it never runs `--full` or `intake import` itself.
- An empty list ends the check.
- An error envelope is neither empty nor non-empty: the orch starts no fork and handles the error as a lyx bug under this section.
- A non-empty list goes to one one-shot fork, unless the orch knows an intake fork is open, in which case it starts no second one.
- An open intake fork is not named in the orch note, since the inbox is its own state: a fork lost to a context cycle leaves its issues open and a later wake's check forks again, and a second fork beside the first is harmless because the CLI settles each issue once (a second import prints `recorded`, a close of an imported issue is refused as closed, an import of a closed issue is refused).
- The fork's bounds, stated as outcomes and pointing to `ly:board` for the commands without restating their mechanics:
  - it reads the full list with `--full` and takes each issue to a new note with exactly one type label and the label of every module the issue touches; a fold into an existing entry only when the issue is essentially the same finding or a correction to its subject, per the board policy, adding any module label the issue brings; or a close as noise, only for an issue that adds nothing to the recorded issue or entry it duplicates or has no actionable content, with a reason naming which.
  - a duplicate that adds anything is folded, and a fold the board refuses because the entry is run-held becomes a new note.
  - when an import's board write succeeds but its close fails, it re-runs the error's `close` payload once, accepting a second pointer comment; a second failure gives the issue a `failed` line with the error and the `close` payload.
  - it writes the board only through `lyx board intake` and a `lyx board upsert` that writes only the labels of an entry it just folded into, keeping every label the entry carries; it edits no file, sends no message, and merges, promotes, starts or re-scopes nothing.
  - it returns one line per issue: the number, then the new slug, the entry folded into, `closed` or `failed`, then the labels, the close reason or the error.
- A `failed` line in the fork's report is handled as a lyx bug under this section, naming the issue and, for a failed close, the `close` payload.

`orch-template-role.md`, After landing: the `landed` bullet's "run `lyx board intake`" becomes a pointer to the inbox check in Where a finding goes.
The other mention, "intake takes those onto the board", stays true and is left.

`SKILL.md`, Intake section: the commands block adds `lyx board intake list --full`.
- `list` prints the number and title of each open issue no entry records, and `--full` adds body, labels, url, comment count and creation time.
- A new note gets its type and module labels through `import`'s `labels`, replacing the line that has module labels added with `upsert` afterwards.
- `import` prints the slug, issue, labels and dropped labels, not the entry; `lyx board get` shows the entry.
- A label `upsert` on a folded entry carries the entry's whole label list, since `labels` replaces it.

**Commit:** 3: The orch checks the inbox and forks intake, and ly:board matches
