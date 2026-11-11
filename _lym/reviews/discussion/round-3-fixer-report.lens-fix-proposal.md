```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the reviewer proposals the fix applied as written (F1 to F5).

Proposals challenged: F1, F2, F3, F4, F5. F4 and F5 hold, so they have no findings below.

1. **MEDIUM** — D4: "When the origin probe fails, the upsert is refused, with the way forward to re-run it once origin answers", and "built on the code `Add` already uses … so `Add` and the upsert cannot disagree".
   - Case: a hub that has no origin, or that `Add` runs with push skipped.
   - `Add` probes origin only when `probeOrigins := !opts.SkipPush && !opts.SkipGit` (`internal/fabricengine/add.go:237`, `:256`), so `Add` never fails on the origin probe there.
   - The upsert's probe runs `ls-remote --heads origin …` (`remoteleftover.go:209`), which fails every time on such a hub.
   - So the upsert is refused forever, its way forward ("once origin answers") can never be met, and it disagrees with `Add`, against the fix's own claim.
   - The function needs `Add`'s skip condition: no origin probe where `Add` makes none.

2. **LOW** — D1's Open risks line: "D6's parent directive has the parent record the call again with `add` first".
   - Case: the run halts at Drift-Verify Stuck for a human, as the tapestry recipe says, and the operator runs `lym loom goto --to Webster`.
   - The parent directive does not reach the operator, so the folded call is still missing from the Master.
   - The line reads as if the directive bounds the gap, but it only covers a goto the parent makes. The line should say an operator goto is not covered.

3. **LOW** — D2: "With no trailer naming the record's path, the history is read whole, as today", which now sits beside a file-history seam that walks first parents only.
   - Today's read uses `PathRevisions`, which walks every parent ordered by committer time.
   - If the new seam is the only history read, then a version reached only through a merged side parent is no longer read, even with no trailer, so "as today" stops being true.
   - D2 should say whether the no-trailer read keeps today's walk or takes the first-parent walk.

Tool calls:
- Read `remoteleftover.go:200-240` and `add.go:250-275`: needed `remoteHeadTip` and how `Add` gates the probe; my context held only lines 80-110 of `remoteleftover.go`.
- Grepped `probeOrigins` and `weftProbeErr` in `add.go`: needed the condition that gates the origin probe, which my context lacked.
