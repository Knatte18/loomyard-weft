# Fixer report — code review round 1

All four findings fixed as proposed, one commit each.
Gate: `lyx gate test ./internal/editdirective ./internal/shellguard ./cmd/lyx` passed.

## Changes

- F1 (MEDIUM), `pattern/PATTERN-refusal-way-forward.md`: removed `sed -i` and `perl -i` from the closed list of denied forms, and removed the in-place spellings clause from the computed-path `rm` bullet.
- F2 (LOW), `internal/shellguard/rows.go`: reworded the `commandPositionPattern` comment so it no longer names the deleted `unquotedPrefixPattern`.
- F3 (LOW), `internal/editdirective/editdirective_test.go`: restored the single "Edit or Write" anchor check, dropped the second phrase and the "never `sed" absence check, and restored the doc comment.
- F4 (LOW), `docs/research/codex-feasibility.md`: dropped `sed -i` from the "Deny guardrails" row.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
