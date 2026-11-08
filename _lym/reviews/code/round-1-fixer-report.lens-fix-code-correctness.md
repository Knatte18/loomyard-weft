```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the `forward` empty-body skip and its new test (fix-code-correctness lens).

No findings.

- **Whitespace-only text:** `"  "` with no files gives `posted == "  "`, so `TrimSpace` is empty and the function returns `true, nil`. Nothing is posted and the cursor advances.
- **Text plus file lines:** `fileLines` is non-empty, so `posted` is non-empty and posts as before. Where the first character is a newline, `TrimPrefix` strips it and `TrimSpace` still sees content.
- **Order of checks:** the empty check runs before the size cap, so an empty body never reaches the over-limit answer or `Post`. The cap rows are untouched.
- **Caller:** `handle` returns `forward`'s `(true, nil)` unchanged. By the semantics `handle` documents, `true` means handled, so the cursor moves past the message. The new test's `wantCursorTS("101.000001")` covers this, and the gate passed it.
- **No `settle()` call on the skip path:** this is correct. The message never reached a `Post`, so it has no failure episode and `failingKey` cannot refer to it.
- **Test against the defect:** without the guard, `Post(orch, "")` is refused by `validateBody` or recorded as an empty mail. The first case fails `mustReadC1`, and the second fails `wantPosts()` with an empty-body mail in the list. So the test fails with the defect in place.
- **Test hygiene:** the new test runs in parallel with its own harness, as the neighbouring tests do.

Tool calls: none. The context already held `inbound.go`, `inbound_test.go` and the gate result.
