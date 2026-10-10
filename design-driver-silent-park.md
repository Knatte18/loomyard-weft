# Driver parks silently on a step failure without kind

`driver-silent-park` · Note · bug

Imported from [issue #488](https://github.com/Knatte18/loomyard/issues/488).

Seen (bolt-and-store, 2026-10-10): at 06:18 UTC `lyx shed step` at Webster-Bouncer returned `ok:false` with `config file /home/knatte/Code/loomyard-LYXHUB/_board/_lyx/config/landing.yaml not found; run "lyx config reconcile"`, no `kind` and no trace.
The driver wrote a drive report, ran `commit-records --park` and told the parent nothing, while the run stayed `running`.
The hub learned of it only through batten's 45-minute "no agent activity" notice, and `lyx loom resume` answered "the run is already running".
The missing file arrived with the hub config reconcile after `gate-test-scope` landed (new `landing.yaml` and `gate.yaml`), so a run started on the older build hit the gap.

Workaround: the orchestrator messaged the driver to step again.

Expected: a step failure without a `kind` is treated as a stop that reaches the parent (a halted run or a driver escalation), never a silent park; a missing hub config file names its cause with a refusal-spec row, and runs in flight across a deploy that adds a hub config file either get it reconciled or are told.
