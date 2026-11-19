```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code in `rows.go`, the new test rows and the regenerated golden, with no source change.

**Findings:** no findings.

I traced these concrete inputs by hand against the new patterns:
- **Single-quoted script, bounded scans.** `bash -c 'claude --version' && echo 'run -p'`, `bash -c 'lym adopt b' && echo 'x --take-hold'`, `bash -c 'sed -n 1p x' ; bash -c 'grep -i foo y'` and `bash -c 'gh api repos/o/r' && echo 'a mutation'` all end the scan at the script's closing quote and are allowed. Under the old scan each crossed the quote and was denied, so every row fails with the defect in place.
- **Script-ending word.** `bash -c 'gh pr view'` is allowed because `scope.ghWordChars()` now excludes `'`. The old class let `view'` count as an unlisted word.
- **Go argument runs.**
  - `sh -c 'go build ./cmd/lym' all` is allowed: the single-scope argument run excludes `'` and cannot reach the outer `all`.
  - `go test -run 'TestA all' ./internal/x` is allowed because the top-level word end drops `'`; with `'` back in the class it is denied again.
  - `bash -c 'go test all'` is still denied, since the single scope's word end includes `'`.
- **F5 gaps.**
  - `sh -c 'echo "$LYM_GATE_SLOT"'` matches the new single-script expansion alternative.
  - `sh -c 'cat <<EOF | rm $d'` and `sh -c 'cat <<EOF > f.go'` match through the new heredoc branches. The top-level alternatives cannot reach them, because their units consume the single-quoted string whole.
  - `bash -c 'grep "LYM_GATE_SLOT" f'` stays allowed: it has no `$`, and the script's units cannot enter the double-quoted string.
- **Parenthesisation.** `goCommandIn` closes `((A)|(B))` correctly, so the position prefix applies to both the `all` and `tags` alternatives. `commandRowPattern` wraps each scope. The `redirectWritePattern` alternation keeps its old precedence.
- **Callers and init order.** `publishCommandPattern()` and the converted `var` patterns are still consumed as `Patterns` strings. Dependency analysis orders `shellScopes` before the vars that reach it through `commandRowPattern`, and `Rows()` runs only from `init()`. The parent's tier-0 gate over the claudeengine, codexengine, configcli and orchcli packages already compiled and passed.
- **`verb_after_a_shell_script`.** `bash -c 'systemctl --quiet' hibernate` fails with the defect in place. The flag word would take the `'`, a space and `hibernate` would follow, and the row would deny.

The rows `rm_after_then`, `zsh_dash_c` and `keyword_as_an_argument` pin behaviour that was already correct; they are coverage rows for F6 and would not fail with a F1/F4/F5 defect. That is expected.

Tool calls: none; the context already held every changed file and the review.
