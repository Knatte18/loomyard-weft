```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens over `decision-record.md`.

**Pairs checked:**

- D1 × D2, the emphasis-line exemption that `TestConfigTemplate` computes from the `review-*` and `fix-*` fans: **broken**. D2 takes `way-forward` out of `review-plan`, so it falls out of `procedureLenses`, and neither D1 nor the Constraint adds it back.
- D1 × Constraint "procedure lenses … end in their own list, every other lens ends with the emphasis line": **broken** for `way-forward`, as above. It holds for `board-scope` and `leanness`.
- D1 × D1 acceptance line "still fails any other lens of a non-procedure fan": **broken**. Under D2, `way-forward` becomes such a lens, and its unchanged prose fails the test.
- D1 × the dormant `discussion` fan: held. `board-scope` is exempt by name, and the fan's other lenses end with the emphasis line.
- D1 × D2, the writer-covers-reviewer check: held. `review-discussion` shrinks to four lenses that are all in `twill-discussion`, and `repo-invariants` is the one reviewer-only lens D2 names.
- D3 × the `csharp` layer and the layer exemption test: held. The exemption is computed from base fans, `numerics` ends with the emphasis line, and the `code-pattern-fit` entry leaves the layer fan.
- D3 × D4, the rubric bullets each lens applies: held. "Test economy" leaves `code-tests` and stays with the lead, and D3 moves its bullets onto `code-affected-set` without touching `code-tests`.
- D5 × D6: held. D6's `""` value needs D5's `CategoryNone`, and D6's blocks pass `checkLocalDocument` once D5 lands.
- D5 × Constraint `PATTERN-config-layers` × D7: held. The background's Guard text changes in the same commit.
- D6 × Constraint on configsync's former-value strip: held as stated. It is not re-checked under this lens.
- D5 × D7, the docs that call the fan keys closed: held.
- D6 × Open risk 1, the live `profiles.yaml`: held.
- D1, D3 × Open risk 2, a hub fan naming a deleted lens: held.

**Findings:**

1. MEDIUM, class `design`, at D2 and at the Constraints lens-prose bullet together with the D1 acceptance line.
   D2 drops `way-forward` from `review-plan` and keeps its prose as it is ("stays defined and stays in `twill-plan`"). That takes it out of the only `review-*` or `fix-*` fan it sat in, so `TestConfigTemplate`'s computed exemption no longer covers it. Its prose ends "Your list: per refusal, the next step, progresses or not." with no emphasis line.
   The Constraint names the procedure lenses as "those of the `review-*` and `fix-*` fans and `board-scope`". D1's acceptance line also requires the test to still fail any other lens that lacks the line. Taken together, they make the unchanged `way-forward` fail the test.
   The record never decides between two fixes: append the emphasis line to `way-forward`, or widen the exemption to name `way-forward` beside `board-scope`, which also means adding it to the Constraint, the header and the acceptance line.
   Evidence: `internal/lenses/template.yaml:288-293`, the `way-forward` prose; `:411-414`, the `review-plan` fan; `:33-36`, the header; `internal/lenses/config_test.go:296-300`, the exemption computed from `wantOrders` `review-*` and `fix-*`; `:341-343`, the emphasis-line check.

Tool calls: none; my context already held everything this lens needed.
