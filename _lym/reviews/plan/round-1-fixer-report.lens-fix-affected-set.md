```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I reran each set-based finding's search and checked that the fix covers every member.

**Unchecked list:** none. F1, F7, F8 and F9 had their searches rerun. F2, F3, F4, F5, F6, F10, F11 and F12 do not rest on a set of sites.

- F1: `ComputeReplies` imports `creelengine` and `agentname`, and the fix adds both to `allowedImports`. The set is covered.
- F7: the only production `LineFits` caller outside `creelengine` is `internal/loomcli/decision.go:304,312` (`noticeFits`), which the fix lists. The set is covered.
- F9: the files under `internal/seatengine` that name `SendMessage` or the session channel are `smoke_seats_test.go` and the two testdata fixtures. All three are now on card 12. The set is covered.
- F8: one member is not covered (finding below). `doc.go:221` sits under `// # Removed holders`, so card 9's doc.go edit covers it.

**[LOW] Card 9 (`_lym/plan/09-sweep-holder-gone.md`), Edit list and the new PATTERN sentence: the sweep run-condition text in code comments stays stale.**
- F8's set is every place that states the sweep runs "while mail or injects exist".
- After card 9, the PATTERN bullet and `doc.go` say the sweep also runs for a read, marked, unanswered message.
- Two doc comments in `internal/creelengine/sweep.go` still state the old condition, and card 9 does not edit them:
  - `sweepLoop` (`sweep.go:36`): "sweeps while unread mail or a queued inject exists". Card 9 lists it only under Uses.
  - `sweepNeeded` (`sweep.go:173`): "reports whether a strand holds unread mail or has an inject queued". It is not on the card at all.
- `sweepSubjects` (`sweep.go:178`) is on card 9's Edit list, so its comment is covered.
- Fix: add `internal/creelengine#Store.sweepNeeded` and `internal/creelengine#Scheduler.sweepLoop` to card 9's Edit list, with one Intent sentence saying both doc comments name the widened run condition.

Tool calls:
1. Grepped production `LineFits` callers, the sweep run-condition wording across `.go` and `.md` files, and seatengine files naming `SendMessage` or the session channel. My context held only the files the fixer had read, not repo-wide search results.
2. Read `sweep.go:34-38` and `171-180`, and the `doc.go` section headings, to confirm the stale comments and which section line 221 belongs to. My context lacked those lines.
