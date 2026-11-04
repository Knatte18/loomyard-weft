---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: plugins/ly/skills/board-policy/SKILL.md:59
    summary: The skill names the decision record "Sketch", a nickname the repository keeps out of files
    fix: In plugins/ly/skills/board-policy/SKILL.md line 59, replace "the part of the Sketch it builds" with "the part of the decision record it builds". The repository's instructions keep the formal name "decision record" in code, files, verbs and specs, and ly:board and the board note already say "decision record".
    origin: handler
  - id: F2
    severity: LOW
    class: consistency
    location: plugins/ly/skills/board-policy/SKILL.md:3
    summary: The skill's description and its INDEX row promise "discuss before writing" as a policy rule, but the body states it only for a session opened from the board button
    fix: In plugins/ly/skills/board-policy/SKILL.md, add one bullet under "## Notes" saying a case is talked through with the operator before it is written to the board, so the orch is bound by it too. Alternatively drop "discuss before writing" from the description at line 3 and from the matching row in plugins/ly/skills/INDEX.md. Keep the description and the INDEX row identical.
    origin: lens:code-affected-set
---

### [LOW] The skill uses the nickname "Sketch" in a shipped file

**Location:** `plugins/ly/skills/board-policy/SKILL.md:59`
**Issue:** The split bullet says each part's body carries "the part of the Sketch it builds".
The repository's instructions say to use the nickname freely with the operator but to keep the formal name, "decision record", in code, files, verbs and specs.
The skill is a shipped file that an agent reads cold.
The note `discussed-task-split` itself says "Code still says 'decision record': nothing in code is renamed to Sketch", and `ly:board` uses "decision record" for the same thing.
An agent reading the skill meets a term no other shipped text defines.

### [LOW] "Discuss before writing" is in the description but not in the policy body

**Location:** `plugins/ly/skills/board-policy/SKILL.md:3`
**Issue:** The `description` and the INDEX row list "discuss before writing" among the policy's rules.
The only matching rule is "Talk each case through with the operator before writing it", under "A session opened for board work", which scopes it to the button's session.
The orch loads the skill and reads the description, yet finds no such rule in the sections that apply to it.
The old orch section did not carry the rule either, so nothing was lost in the move; the description claims more than the body states.

## Policy move check

Every rule the diff removes from `contracts/orchrole/orch-role.md` is in the skill with its meaning intact.
Compared against the orch section at the base commit:
- Notes: proposals go on the board at once, one finding per note, a finding that repeats a note goes into it, the current-design-only body, labels, triage drafts in `.scratch/`.
- Tasks: bundling, never one note alone as a loom task, rename, bench and spike as a darn task with the run cap, Discussion rounds, the profile field, the drift scope.
- Two single rules from outside the section moved with it: never set a priority unless the operator names one, and never set `recipe: darn` yourself.
- The orch role keeps its own rules: decisions on held tasks and beams, the pattern-entry flags, `--amends-plan`, the decision-note fallback and the run lock.
  It also keeps the ban on running an exploration as a background agent of its own.
- `lym board upsert` and `lym board profiles` are gone from the profile bullet, but `ly:board` states both, and the orch loads `ly:board`.
- The orch reaches the policy through `orchSkills`, which both `Start` and the watcher's reload use.
- The skill keeps orch-only tools out: it names no `lym loom decision add`, no batten command and no beam-slug procedure.
  Its "Entries a run holds" section sends a held task's decisions back to the operator naming the orch.
- Every `lym board` verb the skill cites (`set-status`, `get --record`) exists, and the board's task guard refuses a promote only from a session whose strand name carries a task slug.
  The button opens in the prime, where the name has none.

## Scope against the entry

- The skill states promoting notes, releasing a task only on a named go-ahead, and splitting an entry into tasks, with the decision record carried into each part for a discussed entry.
- The button, the repeatable `--skill` flag, the first-turn load and the orch's own load of the skill are all in the diff.
- No board-entry edit, and nothing beyond what the entry licenses.

## Commit messages and change description

Both commit messages and `_lym/landing/summary.md` agree with the diff.
The summary's two open risks, a future Codex `OpenLaunch` that must honour `spec.Skills` and a hub with a custom beam group, are stated as risks and are not defects of this diff.

## Rejected

- Lens code-tests, F-T1 (a refusal row cannot catch a check moved after `OpenLaunch`): the row asserts no strand and no marker directory, and `Runner.Open` runs the check before the token and the marker directory, so a regression that creates either fails it. Whether the fake's `OpenLaunch` ran is speculation about a refactor, and a redundancy or coverage finding needs a named uncovered behaviour.
- Lens code-tests, F-T2 (over-limit row may not assert nothing is created): the shared checks at `command_test.go:487-499` assert the session directory is uncreated and no session id comes back whenever `wantErr` is set, and the row sets `wantErr`.
- Lens code-correctness (an empty `--skill ""` is accepted): it is caller misuse on a non-public surface, and the run path accepts the same input; the code-quality rule is to trust the caller there.
- Lens code-affected-set, finding 2 (a live orch started before this binary lacks the skill until its next cycle): a rollout note, not a defect in the diff; the orch role file and skill list both change at its next launch.
