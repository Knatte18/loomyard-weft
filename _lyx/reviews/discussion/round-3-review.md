---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:97
    summary: D5 migration fires only when all four keys hold the old defaults, so a hub with any pinned key keeps all four old comments, which say empty takes haiku and 0 takes 10 and contradict inheriting shed.yaml
    fix: In D5's migration paragraph (line 97), split the migration in two. The value rewrite stays as written, all four keys at the old defaults together with the old driver_kind comment. Add a per-key comment refresh that runs after it in the same pass. It replaces the comment of any one of the four keys whose comment equals the old template's text for that key with the new template's comment, and leaves the value untouched. A key whose comment was edited by hand is kept. Both steps are idempotent, since the old comment text is gone after the pass. Extend the migration acceptance criterion (line 169) so that a loom.yaml with one key off the old defaults keeps that key and every other value, and each of the four keys carries the new template's comment; a second reconcile changes nothing. If the author prefers to accept the stale comments, add an Open risks line saying a hub with a pinned tackler key keeps comments that contradict inheriting.
---

### [MEDIUM] Migration leaves contradicting comments on hubs with any pinned tackler key

**Location:** `_lyx/discussion/decision-record.md:97` (D5), `_lyx/discussion/decision-record.md:169` (acceptance criterion)

**Issue:** D5 rewrites `loom.yaml` only when all four keys hold the old defaults together with the old `driver_kind` comment, and the acceptance criterion pins the other case as "left untouched".
`internal/loomengine/template.yaml:36-39` shows the old comments are inline on each key: `tackler` says "empty takes haiku", `tackler_timeout_min` says "0 takes 10", and `driver_kind` describes a loom-only seed default.
Once D5 lands, an empty `tackler` inherits `shed.yaml`, and a 0 `tackler_timeout_min` inherits the next layer.
Both comments become false.
`internal/configsync/configsync.go` does not refresh an existing key's comment, as D5 itself says, and this migration rewrite is the only route that replaces one.
A hub that pinned any one of the four keys, say `tackler: sonnet`, is skipped whole, so all four keys keep the contradicting comments for good, and the marker is only ever consumed by a full rewrite.
The drafted comment refresh exists for the all-defaults case and does not reach this one.
The record neither refreshes those comments nor lists the stale comments as a risk.

## Rejected

- Focus 1, D4's bound on a batten tackler's writes: not a defect. `internal/shuttleengine/claudeengine/settings.go` installs only the Agent, AskUserQuestion, python, in-place sed or perl edit, computed-path rm, machine-sleep and raw-go denies, plus the fork-only webster guard. None stops Edit, Write, `git add` or `git commit`, and the tackler spawn sets `Interactive: false` with no further tool restriction (`internal/loomcli/tacklerprompt.go:101`). D4 states exactly that, names the stencil write rule as the only bound, says a resume may re-enter Land or Worktree-Teardown over the state, and records the risk under Open risks together with the push and deploy consequence. The acceptance criterion on the rendered prompt pins the stencil bound the record claims.
- Focus 2, D3's premise scope: not a defect. D3 says the records `shedrun` already declares per run-id keep their paths, lists the drive reports as also written by the LLM driver, the driver lock, the park marker, the resume report and the tackler directory, attempt directories and marker, and restricts the no-migration claim to the settle records. `AppendSettleRecord` is called only from the Go driver (`internal/loomcli/drive.go:282` and `:776`), and the driver log stays a told path. The premise matches the code.
- Focus 3, D4's old-build mechanism: not a defect. The old process holds the run lock through `shedverbs`' generic run body and writes no loop pid file. A new-build `lyx batten run` arming the loop finds no pid file, spawns a detached loop whose step meets the held run lock, and `internal/shedverbs/loop.go` ends it as a busy stop, which `decideDrive` in `internal/loomcli/driverules.go` turns into a busy exit. The waiter's adoption (`internal/shedverbs/doc.go`) needs a held loop lock beside a pid file, which an old-build run never produces, so the record's description of a busy exit rather than an adoption is right.
