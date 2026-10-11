---
verdict: CONTINUE
rationale: "Round 2 carries BLOCKING design finding F1 (D7's SessionEnd path has no launch boundary, so an earlier launch's process-ending SessionEnd ends a resumed launch died); F1 is first raised in round 2 but BLOCKING alone forces CONTINUE. No relabels."
---

Round 2 has one BLOCKING finding, F1 (design), so the loop has not converged.
The remaining findings are F2 (decision, MEDIUM), F3 and F4 (consistency, LOW) and F5 (consistency, NIT); none is gating.

Key mapping:

- F1 is a new key, `d7-session-end-no-launch-boundary`.
- F2 is a new key, `d8-post-paste-confirm-residue-open`.
- F3 maps to the round 1 key `d8-residue-submit-amends-pattern-enter-rule`, narrowed to the background's lead sentence; the never-wipes clause is now amended.
- F4 is a new key, `turn8-ledger-states-superseded-d7-fix`.
- F5 is a new key, `treat-as-empty-rejection-applies-to-d8`.

Round 1 keys not re-raised in round 2 are marked resolved.
The review's focus verdicts show D5 and D7's registry-evidence rule sound, which resolves `d5-escaped-permission-dialog-stays-asking` and `d7-registry-staleness-rule-unsound`.

The review carries no `## Focus departures` section, so there is nothing to ratify or reject.
