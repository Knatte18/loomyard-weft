```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covered every member of each set-premised finding (fix-affected-set lens).

Unchecked findings: none. I searched F1, F2, F3, F4 and F11; F5, F7, F8, F9 and F14 I checked from context, and their named sets are covered.

1. **LOW**: `internal/websterengine/runlevel.go:722` (F4/F11 set, not changed).
   The comment still reads "Merriam may write nothing but its two contract files."
   The stencil, the orch-role table and `audit.go` now all add the friction note. Neither the review nor the fix named this site.

2. **LOW**: `internal/websterengine/state.go:115-116`, the `PreFixHead` doc (F2 set, not changed).
   It says "The gate records it at its first failed evaluation".
   After the F2 change, a restarted run's gate takes over the persisted head and records nothing at its own first failure, because `fail` records only while `preFix == ""`.
   The `NewVerifyGate` doc and `doc.go` were updated for this; the field comment was not.

F1 (records-side wording) and F3 (vetted range, `FixedBase` clearing): no missed members found.

Tool calls (5 of 6):
- `git grep` for write-limit phrasing: my context lacked the repo-wide sites for F4 and F11.
- `git grep` for records-side and "last commit" wording: my context had only the pre-change F1 matches.
- `git grep` for `FixedHead`, `PreFixHead` and fix-commit claims: my context lacked the sites outside the files read (F2, F3).
- `git grep` for `accept-audit` docs and `Git` interface implementers: my context lacked whether contracts or other types needed the change (neither does).
- `git grep` for "pre-fix", plus `runlevel.go:718-724`: my context lacked the reset help and contract descriptions, and the exact current comment.
