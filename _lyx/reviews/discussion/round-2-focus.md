---
round: 2
focus:
  - "D9, the double-notice hold: check that the record names the cause the hold fixes, settled from the code, covering both a long window before the arbiter marker is written (the LLM driver running longer than `noticeReportWait` in `internal/battenshed/notice.go` before the verb in `internal/loomcli/arbiter.go` writes its marker) and a marker that exists but reads not alive in `arbiterInFlightFrom` in `internal/battencli/wire.go`. The question: can a plan writer who never reads the support log act on D9 without finding the cause itself?"
  - "D2 and D1, the upsert-time budget check: check D2's claim about what loom's `ValidateLocalValues` in `internal/loomengine` already checks (today: `driver_kind` and the tackler knobs) against the code, and whether the record decides if loom gains an upsert-time budget check, with its acceptance criteria consistent with that decision."
  - "D5, the shared eligibility function: check that its result for an unreadable escalation record or an unknown cause is stated, and that the verb, the Go driver's facts read in `internal/loomcli/drive.go` and batten's seam in `internal/battencli/wire.go` then route that case the same way."
  - "D4, the `darn.yaml` migration: check that the outcome for a document holding both `review_max_bounces: 0` and an explicit `skip_review` is decided and pinned in the matching acceptance criterion."
  - "Acceptance criteria against D9: check that the hold's start conditions, end conditions and the notice that follows its end agree with D9, including `stateChangeReady`'s `noticeRelayWait` rule for a watched awaiting stop with a parent notice."
---

Focus covers only the areas of the decision record that changed across fixes or hold a gating class; local wording corrections need no focus entry.
