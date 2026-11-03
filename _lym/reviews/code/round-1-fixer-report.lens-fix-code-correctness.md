```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens on commit dd040406, which changed two prose bullets in `loom-rubric-code-review.md`.

The fix changed no Go code, function, branch or signature, so there is nothing to run step by step. It added no test, so no test can pass with the defect in place.

The edit matches the review's `fix` key exactly. Lines 109 and 110 now read "a new assertion on the static prose of a stencil, spec, doc or other text, naming the assertion" and "the static prose of a stencil, spec, doc or other text is exempt". Both match the wording of the plan rubric's widened form, as the review describes it. The `Wording criteria` bullet at line 103 still says a test asserting the prose never stands in for the review check, which agrees with the widened bullets.

Template risk: the file is interpolated as a Go template (`{{.amendment}}`). The edit adds no `{{`, so the template still parses.

No findings.

Tool calls made: none. My context already held the changed file and the review.
