# Card 3 — lyx shuttle open takes --permission-mode

**Edit:**
- `internal/shuttlecli#shuttleCLI.openCmd`
- `internal/shuttlecli#TestOpenCmd_ModelAliasEngine`
- `docs/overview.md`

**Create:**
- `internal/shuttlecli#openRecorderEngine`
- `internal/shuttlecli#openRecorderEngine.OpenLaunch`

**Uses:**
- `internal/testkit/shuttlefake#Engine`
- `internal/testkit/shuttlefake#Reed`
- `internal/shuttlecli#parseSingleEnvelope`

**Intent:** `openCmd` gains a cobra string flag `--permission-mode` (D1, `PATTERN-cli-cobra`), default empty, whose value goes into `OpenSpec.PermissionMode` unchanged; the CLI never inspects it (D2, `PATTERN-shuttle-provider-seam`), so the engine's `OpenLaunch` is the one place a value is validated or refused.
Its help line names the mode words and never a provider flag: "permission mode of the session: bypass or prompt; empty keeps the engine's interactive default".
The `Long` help adds one sentence: `--permission-mode bypass` opens the session without permission dialogs, under its strand name for the role, and an unknown mode is refused before anything is created.
Its example becomes `lyx shuttle open --role explore --model opus --permission-mode bypass` (AC6).

Bound of the new flag (D5): it lets an opened session run every tool without a permission dialog, opt-in per call with `--permission-mode bypass` only; the session still loads Claude Code's own settings layers, the worktree's included, whose deny rules and hooks Claude Code applies in bypass too; it is one interactive session per call in the caller's worktree, refused under a covering pause; it takes no turn until text reaches its pane through the operator in its window, `lyx shuttle send <guid>` (refused under a covering pause), `lyx creel inject` (EXTERNAL callers, programs and orch strands on any strand, any other strand on its own only) or a `lyx creel post` reminder (any strand of the hub), creel holding injects and reminders under a covering pause and its sender boundary being the operator's user through the 0700 lyxtemp root; it carries no shuttle run guard, as an opened session does today; and the opened-session log line records the mode (Card 1).

The shuttle row of `docs/overview.md`, after the sentence on `lyx shuttle open`, adds that `--permission-mode bypass|prompt` chooses the opened session's permission mode, empty keeping the engine's interactive default (AC6).

Test: `TestOpenCmd_ModelAliasEngine` already pins the `OpenSpec` `openCmd` hands `Runner.Open` for its `--model` flag, so it takes the pass-through rows (AC5) rather than a new top-level test.
Both engines of its set, `"fake"` (the default) and `"claude"`, become `*openRecorderEngine` values, so the open reaches `OpenLaunch` instead of being refused for a missing session opener; each row runs `c.openCmd()` with its args and asserts that the row's engine recorder saw exactly one spec, whose `PermissionMode` equals the row's want, and that the other recorder saw none.
Every row but the refusal row asserts through `parseSingleEnvelope` that the envelope is ok.
Rows:
- `--role explore --model opus`: the `claude` recorder, mode empty (the existing alias row).
- `--role explore`: the `fake` recorder, mode empty (the existing default row, and the no-flag case).
- `--role explore --permission-mode bypass`: the `fake` recorder, mode `bypass`.
- `--role explore --permission-mode prompt`: the `fake` recorder, mode `prompt`.
- `--role explore --permission-mode Bypass`: the `fake` recorder, mode `Bypass`, passed unchanged since the CLI never validates.
- `--role explore --permission-mode auto`: the refusal row; the `fake` recorder is scripted with an error whose text holds `invalid permission mode`, and the row asserts through `parseSingleEnvelope` that the envelope is not ok and that its error holds `invalid permission mode`, and that the `fake` recorder still saw exactly one spec, with mode `auto`.
  It keeps the CLI-level coverage of the branch that turns a `Runner.Open` refusal into the error envelope.
Its doc comment changes from naming the refusal to saying it pins the engine and permission mode `open` hands `Runner.Open`, and that a refusal from `Open` reaches the error envelope.
`openRecorderEngine` is a test type in the `shuttlecli` test package that embeds `shuttlefake.Engine` and adds an `err error` field and `OpenLaunch`, which appends the spec it is given to a `specs []shuttleengine.OpenSpec` field and returns a fixed `shuttleengine.Launch{Cmd: "open-cmd", ResumeCmd: "resume-cmd", SessionID: "session-1"}` beside `err`.

**ImpactSummary:** One new flag on `lyx shuttle open` with an empty default, so every existing caller's launch line is unchanged; one doc row and one existing CLI test table change, with no new top-level test.

**Commit:** 3: lyx shuttle open takes --permission-mode
