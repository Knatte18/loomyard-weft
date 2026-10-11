---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lyx/plan/02-fork-activity.md
    summary: card 2 rewrites the field docs but leaves three doc comments that still describe a transcript-only reading, ActivityReader.TurnEndActivity, Claude.TurnEndActivity and the activity.go file header
---

### [LOW] Card 2 leaves three transcript-only activity docs stale

**Location:** `_lyx/plan/02-fork-activity.md`, `**Intent:**` Docs paragraph and `**Edit:**` list.

**Issue:**
D9 changes the activity reading's documented contract from the transcript's last write to the session's last activity, forks included, and card 2's Docs paragraph rewrites `TurnEndActivity.TranscriptModTime`, `AgentActivity.LastActivity`, the `internal/shuttleengine/doc.go` paragraph and the claudeengine `doc.go` paragraph.
Three more comments state the old contract, and the card's Intent says nothing about them:

- `internal/shuttleengine/activity.go:19`, the `ActivityReader.TurnEndActivity` interface method doc: "TurnEndActivity returns the activity of the transcript turnEnd names."
  `internal/shuttleengine#ActivityReader` is not among card 2's targets.
- `internal/shuttleengine/claudeengine/activity.go:22`, the `Claude.TurnEndActivity` doc: "Its modification time is the last write time".
  The glyph is a card 2 target, but the Intent describes only the new body, so an implementer following it can leave the old sentence in place.
- `internal/shuttleengine/claudeengine/activity.go:1`, the file header: "the transcript a Stop payload names through transcript_path is read for its last write time".

After card 2, each of these is wrong for a session whose fork wrote after the main transcript or holds a call open.

**Fix:**
Add `internal/shuttleengine#ActivityReader` to card 2's `**Edit:**` list.
Extend the Docs paragraph so the interface method doc, the `Claude.TurnEndActivity` doc and `claudeengine/activity.go`'s file header each say the reading is the session's last activity, its forks included, with a held fork call reading active up to `bashMaxTimeout`, and say that the API-error part reads the main transcript only.

## Notes on the focus directives

Every directive was followed; none was departed from.

- Card 1, attack surface: the Intent states the bound through the doc text it prescribes.
  Only a call with an explicit `timeout` runs past Claude Code's default, no call stays in the foreground past the ceiling, and at the ceiling the Bash tool ends or backgrounds the call.
  The steer holds no forbidden character, so the `init` check over `standingDenies` keeps passing.
  Package-var initialization orders `bashMaxTimeoutMS` before `steerRawGoDeny` and `standingDenies`.
  The raw-go row is installed in every run, so `TestBuildDenyNotice_SentenceContent`'s non-fork notice reaches the new wording.
  The two literal leads the card names, `command_test.go:71` and `:441`, are the only test lines that spell the env lead out; no test outside claudeengine pins it.
  Not a defect.
- Card 2: the bound is stated and the ceiling cap is reached by the named row, a pending call older than the ceiling reading its write time.
  The rows extend `TestTurnEndActivity`.
  The zero-value `&Claude{}` at `activity_test.go:85` returns before reading the clock, and every other zero-value `Claude` in the tests calls no activity method.
  `corpus_test.go` builds through `New()`.
  `transcriptBlock` already carries `id` and `tool_use_id`.
  Not a defect, apart from F1.
- Card order and glyph forms: card 2 Uses `plan:…#bashMaxTimeout`, which card 1 creates, so the order holds.
  Not a defect.
- Claims edits in cards 3–6: each is a row in an existing `TestStencilClaims` entry, not a new top-level test, so no package moves past its row in `cmd/lyx/testdata/test-budget.yaml`.
  Not a defect.
- Fidelity: D1–D9 and every constraint are carried.
  D5's four parts are in card 4: the step-4 interim line, the fixer interim line, the no-second-fixer line and the corrected held-turn-end sentence.
  D7's `CLAUDE.md` edit is in card 5, and D9's doc contract edits are in card 2.
  `git grep` finds every background-route text outside `_lyx/` and test fixtures in some card's targets.
  No card adds unlicensed scope.
- Re-sign arrows: no signature changes.
  `const` to `var` on `steerRawGoDeny` and `noticeRawGoDeny` changes no parameters, results or receiver, and the field added to `Claude` is a struct change, which takes no arrow.
  Not a defect.
- Verify coverage: `go test ./...` and `go test -tags integration ./...` cover every targeted package, `contracts/stencils` included.
  The section order matches the plan template's.
  Not a defect.
- Card 5: the new way forward still names `lyx gate test`, which the agents' settings allow.
  Every firing row of `TestValidate_VerifyModuleWide` reaches the changed `Detail` through `ValidateFormat`.
  The spec row changes in the same card and commit.
  Not a defect.
