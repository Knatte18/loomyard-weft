---
verdict: APPROVED
prior:
  - id: F15
    status: resolved
    answer: D9 now classifies every key. auto_accept, reviewer_fan, fixer_fan and tree_scope are Guard and hub-file only, and every other key is CategoryNone with D2's bounds. A Guard key in run_settings is refused at seed through HubOnlyWayForward, which internal/configreg/runsettings.go refuseHubOnlyKeys implements. D6 bounds auto_accept, and the acceptance criteria cover the refusal, the loading keys and the red-live escalation. The precedents hold; judge, seed and the model keys are CategoryNone in internal/loomengine/config.go, as D9 has them.
  - id: F16
    status: resolved
    answer: D2 has the Preflight refusal of a package with no test of a live key's tiers, with both ways forward named, and D4 makes a live run that started no test binary read as red. The red mark is harden-live's outcome alone. The go-json counter in internal/gatecli/counters.go already counts per-package tests and fails a package that ran none, so the observation exists. Acceptance criteria are added.
  - id: F17
    status: resolved
    answer: D8's rationale quotes the 2026-10-16 note as written and cites the operator's 2026-11-18 entry as replacing it. No Open risks line on treadle remains, and support-log Q13 is resolved, citing that entry. The earlier arbiter entry is superseded by the later arbiter entry "The 2026-11-18 operator treadle entry supersedes the arbiter's D8 rationale text", so its open-question wording prescribes nothing.
  - id: F18
    status: resolved
    answer: D4's --copies paragraph states the acquireSlot and exe.inherited slot rules, which match internal/gatecli/run.go. The n-slot lease replaces the tier runs' own acquires, so the call cannot wait on a fourth slot. The copies are runs of the key's rendered tier command under its counter, timeout, watchdog and memory guard. The clamp follows gateslot.Pool.AcquireSlots, and the pool-drain bound and the rewritten criterion are present.
  - id: F19
    status: resolved
    answer: D5 has internal/pairteardown spawn lym harden sandbox clear through selfexec.LymBinary before it removes a worktree whose sandbox root exists. It names every route, Finalize, the capped or circling halt, lym fabric remove and Worktree-Teardown, and keeps the worktree with a way forward when the clear fails. The criterion is extended, and fabriccli's remove path does reach pairteardown.
---

No new finding. The round-2 fixes are in place on every changed hunk of `decision-record.md` and `support-log.md`, and each rests on a fact I checked against the code:
the Guard and CategoryNone precedents, the `HubOnlyWayForward` refusal, `AcquireSlots`' clamp and drain, the inherited-slot path, the go-json per-package counter, and burler's fix half, which runs its gates even on a review with no findings, so the live report exists every round.

No `## Focus departures`: every directive was followed.
