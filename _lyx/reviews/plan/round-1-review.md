---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-open-permission-flag.md:27
    summary: Card 3 turns every row of TestOpenCmd_ModelAliasEngine into a successful open, so no test is left that drives openCmd's Runner.Open-refusal branch into its error envelope, the only CLI-level coverage of that path today
    fix: In Card 3's Intent, give openRecorderEngine an `err error` field that OpenLaunch returns beside the fixed launch after recording the spec, and add one row to TestOpenCmd_ModelAliasEngine, `--role explore --permission-mode auto`, whose `fake` recorder is scripted with an error whose text holds `invalid permission mode`; that row asserts through parseSingleEnvelope that the envelope is not ok and that its error holds `invalid permission mode`, and that the `fake` recorder still saw exactly one spec with PermissionMode `auto`. Every other row keeps asserting an ok envelope. The rewritten doc comment says the test pins the engine and permission mode open hands Runner.Open and that a refusal from Open reaches the error envelope. No target changes.
---

### [MEDIUM] Card 3 drops the CLI's only refusal-envelope coverage for `open`

**Location:** `_lyx/plan/03-open-permission-flag.md:27`

**Issue:** `TestOpenCmd_ModelAliasEngine` (`internal/shuttlecli/cli_test.go:739`) is the only test that runs `c.openCmd()`; a grep of `internal/shuttlecli` and `cmd/lyx` tests finds no other.
Today both of its engines are plain `shuttlefake.Engine` values with no `SessionOpener`, so every row goes through the `c.runner.Open` error branch of `internal/shuttlecli/open.go` and asserts the `ok: false` envelope and its error text.
Card 3 makes both engines `*openRecorderEngine` and has every row assert an ok envelope, so after the plan no test reaches the branch that turns a `Runner.Open` refusal into the CLI's error envelope.
That is the branch an operator sees when an unknown `--permission-mode` is refused (AC4 at the CLI surface), and the plan removes its coverage instead of keeping a row for it.
The routing pin itself survives: which recorder sees the spec pins the alias engine as well as the refusal text did.

## Focus notes

- Card 1, log capture: no test in `internal/shuttleengine` asserts the `"shuttle: opened session"` line (grep of the package's tests for the message and for `logcapture` use near `Open` finds none), so the new `permissionMode=bypass` assertion duplicates nothing.
  Dropping `t.Parallel` on `TestRunner_Open` is sound: Go releases a parallel top-level test only after the sequential ones finish, and the serial log subtest runs inside the parent body while the parallel subtests wait for it to return.
  The cited precedent exists at `internal/shuttleengine/agentwait_test.go:79`.
- Card 2 rows against `TestValidatePermissionMode`: not redundant.
  The `auto` row pins that `OpenLaunch` calls the validator at all, which the unit table cannot;
  the `prompt` row pins that `OpenLaunch` passes `interactive=true`, since `prompt` with `false` errors;
  the `bypass` row pins the flag on both lines of `ResumeCmd`.
  The wrong-case `Bypass` refusal is left to the existing unit row, as the card says.
- Card 3 granularity: `openRecorderEngine` and its `OpenLaunch` sit on one card, which is right; the alias row keeps pinning the engine routing.
- Fidelity: D1 to D5, the four constraints and AC1 to AC6 are each carried, including the `buildOpenCmd`, `Claude.OpenLaunch` and `validatePermissionMode` doc comments, both package docs and the `docs/overview.md` shuttle row.
  No card writes a config default, touches `lyx shuttle run` or changes `validatePermissionMode`'s rules.
- Attack surface: Card 3's Intent states the bypass edge's reach and every D5 bound; Card 2 puts the same bound into `Claude.OpenLaunch`'s doc comment.
  Card 1's refusal-before-creation claim is reached by the named "a failed open launch is returned" row, and Card 2's refusal by the named `auto` row.
- Verify: `## verify:` covers `./internal/shuttleengine/...` (which includes `claudeengine`), `./internal/shuttlecli/...` and `./cmd/lyx/...`, in the stencil's order.
- Glyphs: `buildOpenCmd` carries the re-sign arrow with the new `skipPermissions` parameter, and its only caller, `Claude.OpenLaunch`, is on the same card; no other `Edit` glyph changes a signature.
