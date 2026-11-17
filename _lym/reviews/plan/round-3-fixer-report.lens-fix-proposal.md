```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge F1's proposal (two Delete lines for `ownerPause` and `ownerPauseMu`, plus the clause on `ownerpause.go`'s header comment) as card 23 now carries it.

no findings

- Breakage check: no other card in `_lym/plan` names `ownerPause`, `ownerPauseMu`, `ownerpause.go`, `InstallOwnerPause` or `installedOwnerPause`, so no sibling card loses a symbol it relies on. Card 23 is the only one that names them.
- Readers of the deleted vars: no test in `internal/orchcli` reads the vars. `ownerPause` in `internal/loomcli/ownercheck_test.go` is a struct field in another package, so it is untouched.
- File header: `ownerpause.go` still holds `readBeamFacts`, `readOwnRunStatus` and `claimPrimeFor` afterwards, so the header must stay and only drop "the owner pause seam". The proposal asks for exactly that.
- `sync` import: after the deletion nothing uses it. The compiler refuses an unused import, so the implementer has to drop it and it cannot be left behind silently.

Tool calls:
- One grep of `_lym/plan` for every owner-pause name and `ownerpause.go`. My context held only card 23, not the other cards.
