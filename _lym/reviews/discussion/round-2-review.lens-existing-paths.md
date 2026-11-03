```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the record's new state and files through the code that already runs.

Paths traced, per decision:

- **D1 and D2, probe and resume of a retried round:** `probeLiveReview` always probes with `strconv.Itoa(round)`, and the table's Round names every run directory. **Breaks.** A live chair from attempt `<round>b` is never found. The record says only that the assistant's paths come from the recorded token.
- **D1 and D2, recording the attempt token:** `recordRoundFans` runs once, before the attempt loop, so the record has nowhere that writes `<round>b` at attempt 2. **Breaks** (part of F1).
- **D1, removing stale files before the chair starts:** `Call` runs probe, then `archiveStaleOutputs`, then `Review`; the table builder's removal sits after the probe. **Survives.**
- **D2, the `.assist-<seat>.md` copy beside the review:** `highestCompleteRound` reads round files only through `ParseRoundReviewName`, which lens reports already live beside. **Survives.**
- **D2, the copy joining the archive set:** `archiveRound` and the archive before attempt 2 run on the same `paths` slice the copy is added to. **Survives.**
- **D2, the report directory under `/tmp`:** `lymtemp`'s `Root.Sweep` touches only entries under its own root. A reboot clears `/tmp`, and the kept copy under `.lym` covers that. **Survives,** but nothing in the record removes the directories (left to the handler's PATTERN check).
- **D1 and D7, an assistant as a pane-mail strand:** the creel pane path types notices and reminders only for mail addressed to the strand (`internal/creelengine/doc.go:83-95`), and no part of the design addresses one. **Survives.**
- **D5, `config.toml` rendered whole:** Codex writes `config.toml` itself on a trust accept and when `/model` saves a default. **Survives,** and the record states it (D5 Bound).
- **D6, resume line:** reed falls back to the launch line only when the resume line is empty (`internal/reedengine/strand.go:621`); D6's resume line is a non-empty shell exit. A gated run's strand carries no done-when list, so reed relaunches with that resume line and the pane dies. **Survives.**
- **D12, the `codex-file` re-render list:** `target.go:38` gains `CLAUDE.md`. **Survives.**
- **D3, the per-round pick:** `p.models.Pick(round)` is called at both probe and spawn, so the assist pick can share the same index. **Survives.**

Findings:

**F1**
- **Severity:** MEDIUM
- **Class:** design
- **Location:** decision-record D1 (probe and resume), D2 (recorded attempt token, "a probe or resume of the round builds its table from the record"), acceptance criterion "a probe of a retried round finds the retry's assistant".
- **Summary:** Today the review row probes the chair only under round token `<round>` and records fans once before the attempt loop. D2 makes only the assistant's paths follow the recorded token and leaves both of those unchanged.
- **Detail:** After an interruption during attempt `<round>b`, the chair probe under `<round>` finds no live chair. seatengine's Probe then stops only advisors in the table it was handed. The fresh attempt-1 run starts under token `<round>`, so the `<round>b` Codex strand is neither found nor stopped, and the lym process that enforced its timeout has died. That strand keeps spending the capped Codex budget. The acceptance criterion is unreachable while the chair's probe ignores the token.
- **Fix:** D2 says the probe and resume of the whole table, chair included, use the recorded attempt token, and that the token is written at each attempt's spawn, not once before the loop.
- **Evidence:** `internal/shedadapters/burlerreview.go:413-418` (`opts.Round = strconv.Itoa(round)`), `internal/shedadapters/burlerreview.go:229-233` (fans recorded once before the loop), `internal/shedadapters/burlerreview.go:254-259` (attempt token `<round>b`), `internal/seatengine/doc.go:8` (Round names every run directory), `internal/seatengine/doc.go:112` (a probe whose chair is not live stops live advisors).

Tool calls:

1. Grepped `internal/burlerengine` and `internal/shedadapters` for review-stem globs and `opts.Round` use. My context did not show how the attempt token reaches the run names.
2. Grepped `internal` for pane-mail and broadcast delivery. I needed to know whether existing creel paths type into a strand without hooks.
3. Read the pane-mail rules in `internal/creelengine/doc.go`. Same reason as call 2.
4. Read `highestCompleteRound` and the run-dir `ReadDir` sites. I needed to know whether the `.assist-` copy beside the review is misread as a round file.
