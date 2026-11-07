```yaml
lens: code-correctness
outcome: reported
```

Task: check the correctness of the changed code, case by case. I found no defects.

**List (function or branch, the case I ran, correct or not):**
- `FileRaddle` mutation, fresh filing (project plus parts): every part reaches `applyDraftRule` with `released` empty, so all parts are drafts. Correct.
- `FileRaddle`, resume path (`raddleFilingHeld` true): returns `nil, nil` and `filed = slugs`. The old `raddleFiled{}` result was also reduced to `nil`, so the write behaviour is the same and nothing is re-released. Correct.
- `FileRaddle`, `f.Check` error path: unchanged and still returns before any write. Correct.
- `boardCriticalSection`, `raddleFiled` branch removed: `released` is still set at `board.go:138` for the raddle promotion and still passed to `applyDraftRule` at `board.go:148`. No dead or unused variable. Correct.
- `Board.ReleaseTasks` removed: a grep for `ReleaseTasks|raddleFiled|RaddleFile.Release` found no remaining production reference. The only hits are the `raddleFiledSlugs` and `raddleFiledNotice` helpers. Correct.
- `loomshed` `filing()` and `finish()`, beam shape with `CreateBeam` failing: still halts with "make the project a beam", and the notice is not sent. Correct.
- `finish()`, non-beam shape: skips straight to the notice. Correct.
- `finish()`, beam shape with `CreateBeam` succeeding on a retry: the call is idempotent. After the release step was removed, no state is left half-done. Correct.
- `dependencyOrder`: still used at `raddlefile.go:417` by the filing, so it is not dead. Correct.
- `raddleFiledNotice`: `%d`/`%s` args match the format. The landed suffix still appends after the draft clause. A nil `landed` leaves the draft clause only. Correct.
- `Proposal.OrderDepartures` (`omitempty`): `Parse` with the key absent gives nil. With `[]` it gives an empty slice, which is valid per the spec's "absent or empty". Correct.
- `rejectUnknownKeys` over `order_departures`: the slice-of-struct recursion (`proposal.go:181-205`) checks item keys against the `OrderDeparture` tags. So `{"parts","record_order","cost"}` is accepted and an unknown key inside an item is refused. Correct.
- `Hash`: it hashes file bytes, so the new optional key does not change how it is computed. Correct.
- `raddleModeRules(true, …)`: the strings concatenate with trailing spaces and parse as one sentence flow. Auto-mode text is unchanged. Correct.
- The `packages-shared` check: no diff to `check.go`, and no key interacts with it. The refusal of two unordered parts on one package still holds. Correct.

No findings.

**Tool calls (2):**
1. Grep of `board.go` for `released` and of `loomshed` for `dependencyOrder`. My context lacked the post-diff state of `board.go` and whether `dependencyOrder` was still used.
2. Read `internal/raddleparser/proposal.go:150-227`. My context lacked `rejectUnknownKeys`, which I needed to confirm that nested `order_departures` keys are validated.
