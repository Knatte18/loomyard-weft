```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the F1–F4 fixes.

Unchecked findings: none.

No findings.

- **F1:** the search finds no remaining copy of the first-caller rule in `contracts`, `pattern`, `docs` (excluding `docs/research`) or `PATTERN.md`. The remaining "first production caller" hits are allowlist reasons in `cmd/lym/testonlycode_linux_test.go`, not the card rule.
- **F2:** `burler-step-fix.md` is the only site that says what the fixer does with a fact a `fix` names as unverified.
- **F3:** parent consolidation is the only source found where the reviewer carries a change it may not reject.
- **F4:** the carried-finding class rule appears once in `burler-review-format.md`, and the added sentence agrees with it.

Tool calls:
1. `git grep` for first-caller and reachability wording: my context had not searched all of F1's sites after the edit.
2. `git grep` for unverified, disproves and fix-disposition wording: needed the other sites for F2's set.
3. `git grep` for never-reject and carried-finding wording: needed the other sources for F3 and F4.
4. `git grep` over the `burler`, `harden` and `twill` stencils for never-reject, `origin:` and "its `fix`": needed to rule out another source whose findings the reviewer may not reject.
