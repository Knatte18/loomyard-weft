---
round: 2
focus:
  - "D7 (per-row time and visit count in the checklist): settle whether the stated rule matches how `internal/shedengine/run.go` stamps a history entry's `at` (after the producer call returns), how identical stuck calls fold into one entry with `repeats` and `last_at`, how `goto` entries are appended (`internal/shedengine/goto.go`), and that a call with no outcome records no entry; check the acceptance criterion for a row reviewed twice is satisfiable by the rule as written."
  - "D1 and D3 (layout when strands end): settle whether every row of a window is assigned in each reachable state, in particular after `lyx orch stop` removes the orch strand (`internal/orchcli/stop.go`) while the `runs` strand stays, leaving a content strand and Selvage, and in a pair window where every positional strand has ended; the layout must cover the whole window height."
  - "D3 (anchor and size kinds): settle whether each combination of anchor (top, below-parent) and size kind (content, positional, sharing) has a derivable height or a stated refusal, and whether the remainder rows of the even split among sharing strands go to a stated strand, given the byte-identical positional output Constraint."
  - "D8 (hub notice log): settle which batten notices reach the log, given that `internal/battenshed/notice.go` and `rownotice.go` hold a notice back when no orch strand is recorded and drop it after `noticeMaxAttempts`, and whether the stated trigger and its growth bound agree with that."
---

Round 1 left design-class findings on D7 and on D1 with D3; the entries above ask for the underlying questions without presuming the answers.
