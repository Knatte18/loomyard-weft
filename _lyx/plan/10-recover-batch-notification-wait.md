# Card 10 — recover-batch blocks to terminal; Merriam waits only on notifications

**Edit:**
- `internal/websterengine/config.go`
- `internal/websterengine/template.yaml`
- `internal/websterengine/config_test.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/render.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/template_test.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/recoverbatch.go`
- `internal/webstercli/verbs_test.go`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/claims_test.go`

**Intent:**
D6, plus the D3 config key.

Config:
`PollWaitS` and the `poll_wait_s` template key retire;
a hub `webster.yaml` still carrying `poll_wait_s` keeps loading, since the unmarshal ignores an unknown key.
A new `verify_gate_attempts` key (template default 3, validated positive like the other knobs) lands as `Config.VerifyGateAttempts`;
a file without it takes the default through `configengine`'s fill-missing path.
Card 14 reads it.

recover-batch:
a new `websterengine.RecoveryWaitBudget(cfg Config) time.Duration` returns `recovery_timeout_min` plus one poll tick.
`lyx webster recover-batch` uses it whenever `--wait` is 0, so a call returns only at a terminal digest or once `Classify` marks the strand dead on its timeout, since the budget always outlasts the timeout measured from spawn.
`--wait` stays an operator override; its help text and the file's header comment drop `poll_wait_s`.
Bound: `recovery_timeout_min` holds every call, after which the strand classifies dead and the call returns.

Merriam's stencil:
the "started non-interactively" framing goes, replaced by: no operator answers, and ending a turn with no running background work and no outcome file ends the run.
Every failure-ladder rung that calls `recover-batch` now runs `lyx webster recover-batch <NN>` as a backgrounded Bash command, ends the turn, and acts on the command's completion notification;
the re-call-until-terminal rung and the `running`-snapshot handling go, since the call no longer returns one unless an operator passed `--wait`.
The `{{.poll_wait_s}}` marker and its Tuning-knobs sentence go, and `RenderMasterPrompt` drops its `pollWaitS` parameter.
Shuttle reads a turn end with backgrounded Bash outstanding as `EventWaiting`, so the run stays alive with Merriam idle.

`doc.go`'s cold-recovery section describes the blocking call.

Tests:
`config_test.go`: a file carrying `poll_wait_s` loads, a file without `verify_gate_attempts` loads with 3, zero or negative `verify_gate_attempts` is refused naming the key.
`recoverbatch_test.go` (fake clock): `RecoverAwait` with `RecoveryWaitBudget` over a strand that never reports returns a terminal dead/timeout digest, never a running snapshot, and returns at the strand's terminal report when one lands first.
`template_test.go` and `claims_test.go`: Merriam's stencil contains no `sleep`, no `await-batch` or `recover-batch` re-call loop, no "non-interactively", and no `poll_wait_s` marker, and names the backgrounded `recover-batch` call.
`verbs_test.go` and `runlevel_test.go` drop `PollWaitS` from their config literals.

**ImpactSummary:** webster.yaml loses one key and gains one; recover-batch's default wait becomes the recovery timeout; Merriam's failure ladder becomes notification-driven.
