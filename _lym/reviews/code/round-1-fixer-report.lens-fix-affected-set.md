```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Checked:
- **F1 and F2:** a repo-wide search for the old wording ("engine is unknown", "unknown or has no hooks", "hookless", "no hooks, a hand-added") finds no remaining site in `.go` or `.md` files. The two `creelcli/cli.go` help texts (`creel` and `post`) are reworded. `docs/overview.md:476`, `creelengine/doc.go:90` and the `verbs.go` refusal now agree with them.
- **F3:** a doc comment, not a set, so no search applies.
- **F4:** `checkLandingStartConfirmation` has exactly two callers, `session_test.go:702` (`/clear`) and `:789` (`/compact the plan`). Both pass the new `sentText` argument.

Calls made:
1. Grep of the repo for the old F1/F2 wording (`.go`, `.md`): to find stale sites beyond the two the review named.
2. Grep for callers of `checkLandingStartConfirmation` and for "engine unknown" / "hookless" wording in docs, `internal` and `contracts`: to confirm all call sites match the new signature and no other doc repeats the old rule.
3. Grep for "no hooks", "without hooks" and "lym creel read" outside tests: to check the surrounding docs still agree with the reworded text.
