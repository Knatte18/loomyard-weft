# Card 50 — rebaseline-named-cards

**Edit:**
- `internal/websterengine/state.go`
- `internal/websterengine/fingerprint.go`
- `internal/websterengine/rebaseline.go`
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/rebaseline.go`
- `internal/webstercli/validate.go`
- `internal/webstercli/cli.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/fingerprint_rebaseline_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/template_test.go`
- `internal/webstercli/verbs_test.go`

**Prosa:**
- `contracts/stencils/webster/webster-template-master.md`

**Intent:**
Fix PR #321 round-3 finding B's second item: `rebaseline` hashes only begun cards, so an edit to any unbegun card, or to `00-overview.md` whose `## verify:` is the integration gate, is accepted wholesale.
Record a hash of every plan file when the run records its plan, and make the operator name the cards an edit may change.

In `state.go`, `State` gains `PlanFileHashes map[string]string` (`json:"planFileHashes,omitempty"`): the hex SHA-256 of every file the plan fingerprint covers, keyed by file name, `00-overview.md` included.

In `fingerprint.go`:

- A new unexported `planFileHashes(planDir string) (map[string]string, error)` hashes the same file set `fingerprint` reads (every `.md` file but `planparser.AmendmentsFileName`).
- `restampFingerprint` also sets `st.PlanFileHashes`, so every sanctioned plan rewrite by webster itself (handle binding, drift repair, `validate`) moves the baseline with the fingerprint.
- A new unexported `changedPlanFiles(st *State, planDir string) (changed []string, err error)` returns the file names whose current hash differs from `st.PlanFileHashes`, plus files added or removed since, sorted.

In `runlevel.go`, both places `Run` builds a fresh `State` (first init and `--fresh`) set `PlanFileHashes` beside `PlanFingerprint`.

In `rebaseline.go`:

- `RebaselineDeps` gains `Cards []int`, the card numbers the operator names.
- Before the begun-batch checks, when `State.PlanFileHashes` is non-empty, `Rebaseline` computes `changedPlanFiles` and refuses, wrapping `ErrRebaselineCardSetChanged`, when:
  - `00-overview.md` is among them: `00-overview.md changed; it carries the plan's integration verify and is never rebaselined; way forward: restore 00-overview.md, or reset the branch to the run's start commit with git and run "lyx webster run --fresh"`;
  - a changed card file's `NN` number (the digits before the first `-`) is not in `Cards`: name each such file and give the way forward `re-run "lyx webster rebaseline" naming every changed card with --card NN, or restore the unnamed cards`.
- A state without `PlanFileHashes` (written before this change) skips this check, as a record without `CardHashes` skips the begun-card hash check.
- The begun-card checks stay as they are, so a named card that a begun batch holds is still refused when its body changed.
- After the restamp, `RebaselineResult` gains `CardsAccepted []string`, the changed card file names the call accepted.

In `beginbatch.go` and `runlevel.go`, the `ErrFingerprintMismatch` way forward becomes a function of the loaded state and the plan dir instead of the `fingerprintMismatchWayForward` constant:
when `changedPlanFiles` names only card files, it is `run "lyx webster rebaseline --card NN --card MM" to accept the edit, otherwise reset the branch to the run's start commit and run "lyx webster run --fresh"`, with the changed card numbers filled in;
when it names `00-overview.md`, or the state has no `PlanFileHashes`, it names restoring the plan or the `--fresh` route, and `rebaseline` without card numbers only for a state without hashes.
A `changedPlanFiles` error falls back to the generic text, since the refusal must still name a way forward.
`ErrPlanDrifted`'s way forward, and the zero-batch and validation refusals in `runlevel.go`, say `run "lyx webster rebaseline --card NN" naming each card you edited`.

In `internal/webstercli`:

- `rebaseline.go` adds a repeatable `--card` string-slice flag; each value is a card number (`3` or `03`), and a value that is not a positive integer is a usage error naming the value.
  It passes the numbers as `RebaselineDeps.Cards` and adds `cards_accepted` to the success envelope.
  `Long` says the operator names every card the edit changed, that `00-overview.md` is never accepted, and gives the example `lyx webster rebaseline --card 05 --card 07`.
- `validate.go`'s two re-baseline-persist failures drop the `lyx webster rebaseline` alternative and keep `re-run "lyx webster validate"`, since validate's rewrite can touch any card.
- `cli.go`'s verb-list line reads `lyx webster rebaseline --card NN   accept a mid-run edit of the named cards`.

The Master stencil's plan-drift section and done-check section name `lyx webster rebaseline --card NN` where they name `lyx webster rebaseline`.

`doc.go`'s rebaseline paragraph states the named-card rule and the `00-overview.md` refusal.

Tests:

- `rebaseline_test.go`: on the existing begin fixture after batch 1 is recorded done, editing unbegun card 2's Intent and calling `Rebaseline` with no `Cards` → refused naming `02-` and `--card`, `PlanFingerprint` unchanged;
  the same with `Cards: []int{2}` → accepted, `CardsAccepted` lists card 2's file, and `PlanFileHashes` holds its new hash;
  editing `00-overview.md` with every card named → refused naming `00-overview.md` and `--fresh`;
  editing cards 2 and 3 naming only 2 → refused naming card 3's file;
  a state without `PlanFileHashes` → today's begun-card checks only.
  Every existing rebaseline test that edits a card passes that card's number in `Cards`.
- `fingerprint_rebaseline_test.go`: `restampFingerprint` after a card rewrite leaves `PlanFileHashes` equal to `planFileHashes` of the plan dir.
- `runlevel_test.go`: the first-init state records `PlanFileHashes` for every plan file;
  the fingerprint-mismatch refusal after editing card 2 names `lyx webster rebaseline --card 02` and `lyx webster run --fresh`, and after editing `00-overview.md` names `--fresh` and no `--card`;
  the zero-batch and validation refusal assertions expect `rebaseline --card`.
- `beginbatch_test.go`: the plan-drift refusal's error names `lyx webster rebaseline --card NN`.
- `template_test.go`: the done-check section assertion expects `lyx webster rebaseline --card NN`.
- `verbs_test.go`: `TestRebaselineCmd_AcceptsForeignEditAndKeepsRecords` passes `--card` for its edited card and asserts `cards_accepted`;
  a new `TestRebaselineCmd_RefusesUnnamedCard` edits a card, passes no `--card`, and asserts a non-zero exit naming `--card` with `state.json` byte-identical;
  `--card x` exits non-zero naming `x`.

**ImpactSummary:** Run state records a hash of every plan file, and rebaseline accepts an edit only to the cards it names, never to 00-overview.md; every refusal that names rebaseline now names the cards to pass.
