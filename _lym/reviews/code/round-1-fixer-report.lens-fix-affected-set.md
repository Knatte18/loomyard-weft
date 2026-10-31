No findings.

Coverage checked: every site that describes the resume reconcile already states the Worktree-Teardown pair-gone exception.
- `internal/battencli/cli.go:64`, `arm.go:713`, `owner.go:230-233` and `docs/overview.md:704` already carry it.
- The fixer's additions at `pattern/PATTERN-owner-machine.md:21` and `:28` and `internal/battenshed/doc.go:60` now match them.
- `PATTERN-owner-machine.md:3`, the header, does not name the new case, but it is not a claim that every resume reconciles. LOW at most, below the bar for a finding.
- `PATTERN-owner-machine.md:29`, the origin-owed marker bullet, says a resume that judged its owner without origin leaves the marker. The pair-gone skip writes none, so the bullet and the new exception at `:21` disagree. LOW, since `:21` states the no-marker behaviour explicitly.
- `pattern/PATTERN.md` does not exist, so the F2 index check found nothing there. The index may live elsewhere; not searched.

Unchecked: F1's test-row set and F4/F5.

Calls made:
1. `grep -rnIi` over md and go files for resume/origin-reconcile statements, to find every site stating an unconditional resume reconcile that my context lacked.
