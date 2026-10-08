---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md (D1)
    summary: D1 does not decide whether record-batch resolves head_sha before its correctness-failure path, which records the raw report head in the failed digest, nor what an unresolvable abbreviation does on that path
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D4)
    summary: D4 subtracts the texts readRunStartBase reconstructs (raw PATTERN.md and raw Master template, no CLAUDE.local.md), which are not the texts MerriamBase computes or the measurement point holds, so the derived fixed figure absorbs the directive wrapper and any CLAUDE.local.md that MerriamBase then counts again
---

### [MEDIUM] D1 leaves the head_sha resolution point open relative to record-batch's correctness-failure path

**Location:** `_lyx/discussion/decision-record.md`, D1 (Decision and Rationale).

**Issue:** D1 lists what the resolved SHA replaces downstream (`reconcileHead`'s walk, the digest's `HeadSHA`, `CardSHAs`, the done-check and drift delta end) and its Rationale claims "the stored run record never holds an abbreviation".
In `RecordBatch` (`internal/websterengine/recordbatch.go`), a correctness finding fails the batch before the report head is reconciled: the branch at `len(correctness) > 0` parses the report and hands `r.HeadSHA` straight to `failOnCorrectness`, which passes it to `failFromFindings`, which writes it as the failed digest's `HeadSHA`.
Unless the resolution runs on the parsed report before that branch, a correctness-failed batch with an abbreviated `head_sha` stores the abbreviation, against D1's own Rationale.
And if it does run there, D1 does not say what an abbreviation that resolves to no commit or to several does on that path: refuse with D1's new refusal (leaving the correctness finding undispositioned and the batch non-terminal), or fail the batch on its finding with an empty or raw head.
This is a behavior choice, not a call-site list: the order of "fail on correctness" versus "refuse an unresolvable head" decides whether a bad abbreviation can hold off a correctness failure.

**Fix:** state in D1 that the helper resolves the parsed report's `head_sha` once, before any use of it in the verb, the correctness-failure digest included, and pick what an unresolvable value does when the batch also carries a correctness finding (for example: the correctness failure wins and the digest records an empty head, or the D1 refusal wins).
Add an acceptance criterion for the chosen case.

### [MEDIUM] D4's subtracted texts differ from the texts the base computes and the measurement point holds

**Location:** `_lyx/discussion/decision-record.md`, D4 (Decision and Rationale).

**Issue:** D4 subtracts "the texts the base computes, reconstructed at the run's base commit as `readRunStartBase` does but without `00-overview.md`", and its Rationale says "the subtraction removes only lines present there".
`readRunStartBase` (`tools/tokencount/calibrate.go`) reconstructs `CLAUDE.md`, the whole `PATTERN.md` and the raw Master template (`masterTemplatePath`).
`MerriamBase` (`internal/websterengine/merriambase.go`) computes a different set: `CLAUDE.md` and `CLAUDE.local.md`, the Master template, and `pattern.Directive(..., RoleOrchestrator)`, which is `PATTERN.md`'s overview filled into the `pattern-directive-orchestrator` stencil (20 lines in the hub's stencil store).
At D4's measurement point Merriam's context holds the rendered prompt file, so the directive wrapper's lines and any `CLAUDE.local.md` are present but never subtracted; `CLAUDE.local.md` is untracked, so no reconstruction at a base commit can recover it.
Those lines land in the derived fixed figure, and `MerriamBase` then counts them a second time through its own texts, so the calibrated constant reproduces the measured start only on a tree with no `CLAUDE.local.md` and only up to the wrapper's lines.
The figure D4 claims to measure ("system prompt, tools and the skills loaded at launch") is therefore not what the derivation yields.

**Fix:** have D4 subtract exactly the texts `MerriamBase` counts, minus `00-overview.md`: the orchestrator directive as rendered at the base commit (the directive stencil filled with `PATTERN.md`'s overview), not the raw `PATTERN.md`.
Decide how `CLAUDE.local.md` is handled: either the section skips or flags a run whose worktree had one, or the Decision states that it cannot be reconstructed and is absorbed, with the bound on the error.
Adjust the acceptance criterion on the fixture run to pin the subtracted line count.

## Focus verdicts

- D2, carried-over count: a report that appears later with no bracket transcript showing a write to the batch's report path is still `ErrNoForkTranscripts`, and D2's Rationale states the bound (only a report its own fork wrote with a tool the audit sees); the acceptance criterion "After a no-report call, a report that no bracket transcript shows a write to is still archived" pins it. Not a defect.
- D2, re-fork inside the open bracket: D2 settles `SettleRetry` against the full `SeenForkTranscripts` and only widens the fetch's seen set, so the settle window still waits for the new fork's transcript; an acceptance criterion pins it. Not a defect.
- D2, re-begin: D2 states that every begin-batch clears the bracket's list even where it keeps `StartSHA`, and an acceptance criterion pins zero new transcripts after a re-begin. Not a defect.
- D4, launch sequence: `loadSkillsThenPrompt` (`internal/shuttleengine/run.go`) types the skill load turn before the prompt line, matching D4's measurement point, the auto-mode assumption on the first Read, and the absence of a contradicting open risk. The subtraction itself is F2.
- D4, several sessions: D4 measures the first Merriam session by start time and lists the others as skips with the reason. Not a defect.
- D1, route: D1 and the Constraints both pin `git rev-parse --disambiguate` through `gitexec.Run` and exclude `gitrepo.Repo.ResolveSHA`. Probed: `--disambiguate` lists every object for a prefix, exits 0 with empty output on no match, accepts uppercase hex, and lists objects only, never refs. Not a defect.
- D1 and D5, non-hex fallback: a non-hex value reaches `reconcileHead`'s mismatch refusal in record-batch and recover-batch, and `gitrepo.SHAExists` returns false for it (`validSHA` requires 4 to 64 hex), so `reset --to report-head` meets its missing-commit refusal, as D1 says. D2 states the new `ErrNoForkTranscripts` trigger and D5 points at it rather than restating it; the ambiguous-abbreviation case has an acceptance criterion. Not a defect.
- Relocation: `decision-record.md` holds no session ids; the remaining commit hash (`e003ef6d9`) and the `batcher.yaml` line sit in Scope's Out list as scope boundaries, and the rejected `batcher.yaml` alternative's deliberation is already in `support-log.md`. No relocation finding.
