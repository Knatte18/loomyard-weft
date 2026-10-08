---
round: 1
focus:
  - "internal/shuttleengine/send.go: does every verified send path (start-time, gate, re-prompt) go through the idle wait, settle, confirm and own-text clear in the order cards 3 and 4 name, and are all timed waits read through the `sendContext` `Clock` rather than a real sleep?"
  - "internal/shuttleengine/send.go and run.go: a send that does not land must fail `ErrSubmissionNotLanded` and clear only the shuttle's own typed text; settle whether a partial or paste-placeholder state can leave text behind or clear text the shuttle did not type."
  - "internal/shuttleengine/run.go and wait.go (cards 5 and 6): a start-time send or persist failure tears the run down as not started, while an unlanded gate re-prompt retries on a later tick without spending an attempt; settle whether either branch can halt a run, spend an attempt or leave a strand behind."
  - "internal/shuttleengine/stop.go and its callers in attach, burler and webster (cards 7 to 10): `stopped` is recorded before the strand is removed on every stop path; settle whether any caller still removes a strand without recording, and whether the deleted removers leave no dangling reference."
  - "Per-card mechanical check: for each Delete group (card 10) and Rename group in the plan, confirm the type-specific check from the loom plan spec ran against every target of that group, not only the first."
  - "Test economy: the large additions in run_test.go, startup_test.go, gate_test.go and stop_test.go; settle which new tests duplicate coverage another named test already gives, and which added behavior has no coverage."
  - "Docs landing with the code: `doc.go` of shuttleengine and websterengine, `PATTERN-shuttle-stop` and the `PATTERN-completion-signal` edit match the behavior the diff implements, and the new config keys in template.yaml match `LoadConfig` validation."
---

Round 1 seed from the plan's card index; no prior findings exist.
