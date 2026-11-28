---
round: 1
focus:
  - "Trace every path on which an orch reads as down, in `lym orch status` and in the routing, and settle whether each names its cause. Include a worktree with no reed session, where `internal/orchcli/status.go` takes `c.strands.Strands()`, and errors returned before `orchroute.ReadingOf` is reached. Check the verb's help list of `strand_down` values against what the verb can emit."
  - "Settle whether `strand_live` and `strand_down` in one `lym orch status` envelope can disagree. They come from separate computations (`trackedStrand` in `statusFields`, and `orchroute.ReadingOf`), over a state and strand list read before a possible watcher restart."
  - "Check `downCause` in `internal/reedengine/apply.go` against how `Status` builds each pane set. Does every non-live strand get the right cause, including an empty pane id, a dead pane in another session, a failed cross-session lookup, a stale pane binding and Windows? Can a live strand carry a cause, or a non-live one an empty cause? Which of these cases do tests cover?"
  - "Check how often `orchroute.Live` is called (the beam-chain walk, creel, `lym beam status`, any polling loop) and what its new Info line costs on those paths. Also settle whether a cause logged at Debug for a never-started orch meets the record's demand that the routing's log line name the cause."
  - "Check the claims in `_lym/landing/summary.md` and the commit `Root-Cause:` trailer against the diff and the amended record. This includes what the summary says about issue #232, who reads through `orchroute.Live`, the routing being unchanged, and the Merge danger blast radius."
  - "Check that doc comments, command help and any repository doc describing `lym orch status` fields or reed strand status match the new `Down` field and `strand_down` key. Check `PATTERN.md` and `pattern/` for rules that apply to a new envelope field or log line."
---

The record's one `Added after Discussion` entry settles the scope: name the cause of every down reading, and do not change the liveness rule.
