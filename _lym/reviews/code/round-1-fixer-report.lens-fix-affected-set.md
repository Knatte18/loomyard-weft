```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set-premised searches for F6 (docs on the fixer keys) and F5 (callers of the dirty-tree reason) and check the fix covers every member.

**Findings:** no findings.

**Unchecked findings:** F1–F4 are single-test coverage gaps, not sets; I did not search them.

- **F6:** the grep for `fixer_timeout_min`, "merge fixer's" and "merge fixer session" over `.md`, `.yaml` and non-test `.go` files finds no other place that still says the keys serve the merge fixer alone.
  - `contracts/specs/llm-model-spec.md:128`, `docs/overview.md:585`, `internal/landingshed/template.yaml:11-12` and `internal/landingshed/config.go:56-59` all name the fix review session.
  - `internal/landingshed/fixreview.go:115` ("runs on the merge fixer's model-spec and turn timeout") and `internal/landingshed/fixerrun.go:49,248` are accurate as they stand.
  - The doc.go bound at `internal/landingshed/doc.go:58` already names `fixer_timeout_min`.
- **F5:** the other callers of `dirtyReason` and `clean()` keep the unchanged "commit or remove them on the task branch, then resume" tail.
  - Those are `parentalign.go:168,192,201`, `finalize.go:625,839,859`, `fixerrun.go:250` and `verifygate.go:274`. In each, a commit on the task branch is the intended way forward.
  - Only `fixreview.go:260` trims the tail with `dirtyWayForward`, which is the one site where a commit would move the head past the judged range.

**Tool calls:**
1. Grep for the fixer key and merge-fixer phrases over `.md`, `.yaml` and non-test `.go` files: my context had only the two doc sites the review named.
2. Grep for `dirtyWayForward`, `dirtyReason` and `.clean(` over `internal` Go files: my context did not hold the full caller set of the dirty-tree reason.
