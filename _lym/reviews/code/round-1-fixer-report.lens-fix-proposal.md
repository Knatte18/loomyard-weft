```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1–F6) and build counter-cases.

**LOW** — `internal/landingshed/doc.go` and the `mergeOnGitHub` doc comment: "Finalize halts within about two minutes" understates the worst case.
- Evidence: the retry gate is `pr.Now().Before(deadline)` at `finalize_pr.go:510`, and the sleep follows it. An attempt that passes the gate at about 59s sleeps up to 32s, merges at about 91s, and is refused. The following `ReadMergeState` can then poll `unknown` for up to a further minute, which puts the halt at about 151s.
- The review's own wording ("about two minutes") has the same slack, so this is the proposal's imprecision and not a fixer slip.
- A reader sizing the parent lock hold from "about two minutes" would be about 30s short.
- Suggested fix: say "up to about two and a half minutes", or say that the bound is the window plus the final backoff plus one reading.

No other finding:
- F1: `MergeabilityUncomputed` (`State` open, `MergeableState` `unknown`, exactly one cause) cannot misfire.
  - A draft, a head mismatch or a closed pull request each adds a cause or fails the `State` check.
  - A `RefusalOther` with a real approval or check cause cannot sit under `unknown`, because `mergeCauses` yields only the uncomputed cause for it.
  - The beam wording keeps the status verb and omits the cancel, which matches F4's documented exception.
  - The new `wantNotInReason` entries are on the task row, where they would fail with the defect in place.
- F2: the new row ends at `Status 7`, which implies no second merge.
- F5 and F6: the text matches the code. `mergeCauses` is read by `internal/beamcli/pr.go:375` and `internal/beamcli/land.go:349`.

Tool calls made: none; my context already held every changed file.
