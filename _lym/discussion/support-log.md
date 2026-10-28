# Support log — webster-recovery

## Interview

A darn run holds no interview; the board entry and the archived refusal notes of task `umbrella-anchor` are the inputs.

## Rejected alternatives

- Admit `lym gate test` in `IsReadOnlyCommand`: it only downgrades the finding to policy, and the incident's chained `git add && git commit && … lym gate test` command would stay correctness, so the batch would still fail.
- Have recover-batch accept in line on the at-start basis too: that basis leaves the fabric repo's state for the caller to vouch for, which an automatic acceptance would do without any caller choosing to.
- Allow `reset --to batch-start` while the run lock is held: the Master stencil permits no reset but `report-head`, and the reset discards a batch's work before the parent has judged it.
- Name the matched weft text in the finding's detail: changes the `RefMatcher` seam and the detail format for a case D1 already stops; left for a follow-up.

## Review rounds

_No rounds yet._

## Question ledger

- Q1: Which rule flagged the gate command? Self-pick: the bare weft-name rule, on the `--card` value ending in `-weft`; resolved from the archived record-batch note.
- Q2: Exempt the gate segment from the bare rule only, or from every rule? Self-pick: the bare rule only, so a real weft path in a gate command still matches.
- Q3: Keep recover-batch's refusal as `needs_fresh` when the evidence check refuses? Self-pick: yes, since Master's ladder handles it and the batch still cannot be recovered in line.
- Q4: Fix the reset by allowing it in-run or by naming who runs it? Self-pick: name who runs it, per D3.
