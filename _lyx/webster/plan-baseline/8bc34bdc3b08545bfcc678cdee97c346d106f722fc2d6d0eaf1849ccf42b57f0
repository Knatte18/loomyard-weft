# Card 3 — denied-forms-from-table

**Create:**
- `plan:cmd/lyx#operatorDeniedForms` -> `var operatorDeniedForms [][]string`
- `plan:cmd/lyx#shellGuardForms` -> `func shellGuardForms() [][]string`

**Edit:**
- `cmd/lyx#deniedRecoveryForms`
- `cmd/lyx#TestNoDeniedRecovery_ScanSelfCheck`
- `pattern/PATTERN-no-denied-recovery.md`

**Uses:**
- `internal/shellguard#Rows`
- `internal/shellguard#Row`

**Intent:**
Make the no-denied-recovery scan read lyx's own denied forms from the shell-guard table instead of mirroring it by hand (D7).

`operatorDeniedForms` holds the operator's permission-set forms no lyx table owns: `git reset --hard`, `git push --force`, `git push -f` and `rm -rf`.
`shellGuardForms` returns the `Forms` of every `shellguard.Rows("")` row in table order; the foreground phrase does not touch any form.
`deniedRecoveryForms` becomes `operatorDeniedForms` followed by `shellGuardForms()`, and its comment names the two owners: `internal/shellguard` for lyx's own denies and the operator's permission set for the rest.
The scan, its scanned surface, its `never ` marker and its token matching are otherwise unchanged.

`TestNoDeniedRecovery_ScanSelfCheck` keeps its planted rows and gains one planted row per entry of `deniedRecoveryForms`, `way forward: <form>` expecting exactly one hit naming that form, so the self-test still plants and catches each form and a form the table adds is caught without a hand edit here.
Since `deniedRecoveryForms` is built as `operatorDeniedForms` followed by `shellGuardForms()`, the scan's forms equal the operator list plus the table's forms by construction, and the per-form planted rows prove the scan catches each.

`pattern/PATTERN-no-denied-recovery.md` names `internal/shellguard` as the owner of lyx's own denies in place of `claudeengine`'s `settings.go`, says the list in `cmd/lyx/norecoverydeny_test.go` reads the table's forms and lists the operator's, and its raw `go` bullet says the raw `go` deny in `shellguard`'s table, where it said `claudeengine`'s.

**ImpactSummary:** Test-only change in `cmd/lyx` plus one background file; the scan's form set is the same as before, now derived from `shellguard`.
