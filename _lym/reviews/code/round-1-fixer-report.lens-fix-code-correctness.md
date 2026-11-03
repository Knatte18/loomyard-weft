```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the code correctness of the fix for F1.

no findings

The fix changed one line of prose in `docs/overview.md` (commit 9544be34). It added no function, branch, signature or test, so there is no code path to run, no caller to recheck and no test that could pass with the defect in place. The added clause matches the wording in `internal/boardengine/doc.go` (line 206: "with a `Sketch` link to its decision record file under it", line 294: "links it as `Sketch`").

Further tool calls: none. My context already held the edit and the `doc.go` grep output.
