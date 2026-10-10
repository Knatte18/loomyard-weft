---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/02-classifier-shell-words.md
    summary: Card 2 bounds unquoted globs only inside gitForm.admits, so with plainWord deleted the branch --list patterns, cat-file and config positionals and the -C directory admit an unquoted glob that D2 rejects
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/plan/08-thin-table-rows.md
    summary: Card 8's awaited-shell row expects EndedShells empty on the ungated at-once finish, but that finish records every outstanding shell, awaited or not, so the row fails against the unchanged code
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/02-classifier-shell-words.md
    summary: Card 2 adds classifier rows the table already holds (git log a?, git log [a], git log {a,b}) and lists git log ~/x both as a flipped existing row and as a new admitted row
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/04-own-json-flag-pin.md
    summary: Card 4 says cmd/lyx sits at its budget row, but the package holds 71 top-level tests against a row of 72, so the new pin fits and raising the row loosens the budget
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/06-merriam-base-one-way-forward.md
    summary: Card 6's ImpactSummary says four call sites gain their own way forward, while its Intent changes three and leaves autoRebaseline unchanged
---

### [BLOCKING] Unquoted globs stay admitted outside `gitForm.admits`

**Location:** `_lyx/plan/02-classifier-shell-words.md`, the `admits` and subcommand-word paragraphs.

**Issue:**
Today `plainWord` (`internal/fabricengine/refscanner.go:263`) rejects `*`, `?` and `[` in every word of a `git` segment, and Card 2 deletes it.
The card's only replacement glob check is in `gitForm.admits`: it "rejects an unquoted-glob word before `--` everywhere and after `--` where the field is unset".
`admits` covers only the `gitForms` entries and `remote get-url`/`remote show`.
Four paths never call it, and the card gives them no glob rule:

- `readOnlyGitBranch`'s `--list` path, which checks only for a leading `-` (`refscanner.go:417-419`), so `git branch --list *` would pass;
- the `cat-file` object word and the `config` key and value words (`refscanner.go:388-391`), which also check only for a leading `-`;
- the `-C` directory, which the card requires to "carry no quote" but never to hold no glob, so `git -C * status` would pass.

D2 admits an unquoted git glob "only in a word after `--` in `log`, `show`, `diff` and `status`".
In these four places the shell expands a glob before git sees the words, so a planted name spelling a flag can reach git, which is the attack D2 bounds.
The card's Intent lets this through without stating it, which the rubric's attack-surface rule forbids, and no rejected row pins any of the four.

**Fix:**
State in Card 2's Intent that every word of a `git` segment that `admits` does not judge rejects an unquoted glob: the `-C` directory, `branch --list` patterns, `cat-file` and `config` words.
Add rejected rows to `TestIsReadOnlyCommand`, for example `git branch --list *`, `git -C * status` and `git cat-file -p *`.

### [BLOCKING] The awaited-shell row asserts something the wait never does

**Location:** `_lyx/plan/08-thin-table-rows.md`, the second new row of `TestWait_RecordsEndedShells`.

**Issue:**
The card plans a row "on the ungated at-once finish" with an awaited shell alone, expecting `EndedShells` empty.
The table's runs carry no gate (`gateConfig` at `internal/shuttleengine/fixture_test.go:27` is a `Config`, not a run gate).
For an ungated run, `countsAtOnce` returns true whatever is outstanding (`wait.go:766-769`).
`expiredTurnEnd` then moves the waiting tasks into `countedTasks` (`wait.go:762`), and `endedShells` records every `BackgroundShell` among them with no awaited filter (`wait.go:816-829`).
The awaited shell therefore lands in `EndedShells`, and the row fails.
Card 8 changes no production code, and the decision record licenses rows here, not a change to shuttle's record.
So as written the card cannot carry the acceptance criterion "an awaited shell never lands there".

**Fix:**
Pin the criterion where the code holds it: a gated, fresh run whose awaited shell makes `countsAtOnce` false, so the waiting turn end does not count, followed by a turn end with nothing outstanding that finishes Done with `EndedShells` empty.
This may be a row in this table with a gate column, or a row in `TestPollEventsTick_OutstandingBackgroundWork`, which already scripts the awaited spec.
If no existing table can drive that sequence, the card says so and the fixer disputes the criterion rather than planning a row that fails.

### [MEDIUM] Card 2 duplicates existing classifier rows

**Location:** `_lyx/plan/02-classifier-shell-words.md`, the rejected and admitted row lists.

**Issue:**
`TestIsReadOnlyCommand` already holds "git log with a glob question mark" (`git log a?`), "git log with a glob bracket" (`git log [a]`) and "git log with a brace" (`git log {a,b}`) (`internal/fabricengine/refscanner_test.go:200-203`), and none of them changes outcome under the new rules.
Card 2 lists all three again as new rejected rows.
It also says the existing "git log with a home path" row (`git log ~/x`, `refscanner_test.go:202`) flips to admitted, while listing `git log ~/x` among the new admitted rows.
`PATTERN-test-economy` makes a redundant row a defect, and Card 5 shows the plan knows the rule: it declines to repeat the `log` and `ls-remote` rows.

**Fix:**
Drop `git log a?`, `git log [a]`, `git log {a,b}` and the new `git log ~/x` from Card 2's row lists, and keep the existing rows, the home-path row reworded as it flips.

### [LOW] Card 4's budget premise is false

**Location:** `_lyx/plan/04-own-json-flag-pin.md`, last Intent paragraph, and its `cmd/lyx/testdata/test-budget.yaml` Edit target.

**Issue:**
`cmd/lyx/testdata/test-budget.yaml` pins `cmd/lyx: 72`.
Counted the way `isTopLevelTest` counts, with `TestMain` excluded, the package's `_test.go` files hold 71 top-level tests, so `TestOwnJSONFlags` brings it to 72, inside the row.
The card says the package "sits at its row" and raises the row by one, which leaves one unused test of headroom.
CLAUDE.md requires the budget file under `Edit:` only when a card's new tests push a package past its row.

**Fix:**
Drop the budget edit and its sentence from Card 4, and leave the row at 72.
If a parallel landing in `cmd/lyx` has used up the slot by implementation time, the card raises the row then.

### [LOW] Card 6's ImpactSummary miscounts its call sites

**Location:** `_lyx/plan/06-merriam-base-one-way-forward.md`, `**ImpactSummary:**`.

**Issue:**
The summary says "four call sites gain their own" way forward.
The Intent gives one to three sites: `Run`'s fresh partition, `websterCLI.executionBatches` and `websterCLI.rebaselineCmd`.
It says `autoRebaseline` "is unchanged", and the code has exactly these four callers of `MerriamBase` (`runlevel.go:461`, `autorebaseline.go:49`, `cli.go:298`, `rebaseline.go:98`).

**Fix:**
Change the summary to say three of the four call sites gain their own way forward, and that `autoRebaseline`'s existing transient wrapper already supplies one.

## Verdicts on the focus directive's other questions

- Fidelity: D1 through D7 and every Constraint are carried.
  The shared decision that Card 4 pins the walk's actual set is licensed: D3 orders the set the walk reports pinned, and "today `lyx shed status` alone" is a factual claim the card corrects.
  `internal/shedverbs/status.go:230` declares the local `--json` under every shed mount.
  The rows-only decision of Card 5 matches D4 and the auto-mode assumption.
- Verify coverage: `go test ./...` and `go test -tags integration ./...` cover every targeted package, `internal/shuttleengine` included, and the section's order matches the plan template.
- Edit-card glyphs and re-sign arrows: Cards 1, 2, 3, 6 and 7 list member glyphs, and the arrows sit where a signature changes (`readOnlySegment`, the git segment rules, `admits`, `countPositionals`, `existingReportRemedy`).
  `merriamBaseError`'s and `BeginBatch`'s signatures are unchanged, so no arrow is missing.
- Card order: no card uses something a later card creates.
  Card 3 edits `readOnlyLyxSegment` after Card 2 retypes it, and Card 5's rows follow Card 2's word type.
- Attack surface: Card 3 states its `--json` bound, Card 7 states the `recovery_exhausted` flag's bound, and Card 5 admits nothing new.
  Card 2's gap is F1.
- ImpactSummary: every `Edit` and `Delete` card carries a blast-radius conclusion, with Card 6's count wrong as F5 states.
