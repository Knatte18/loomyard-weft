```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the F1–F3 change set.

No findings.

- **F1:** `internal/dotnetimpact/doc.go:7` says verbatim `@"…"` strings use doubled quotes and raw strings use `"""…"""`. The `!verbatim` guard matches that. The `lex.go:179` doc comment and the guard agree: a verbatim string now takes the quote-doubling loop whatever its opening run.
- **F2:** `typeDecl` still calls `declaredName` itself. It reads `p.tokens[p.i+1]` only after `startsTypeDecl` has confirmed an identifier follows, so the two don't disagree. No doc text covers the `record` identifier case.
- **F3:** `internal/gatecli/doc.go:71` already says a narrowed project invocation writes no entry and only whole entries carry. The new text at line 181 matches it.

Calls made:
1. grep over `internal/dotnetimpact/doc.go`, `lex.go`, `docs` and `internal/gatecli/doc.go`, because my context lacked the package doc and the other gatecli doc lines that could repeat the changed claims.
