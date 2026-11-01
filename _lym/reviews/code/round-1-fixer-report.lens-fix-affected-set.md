```yaml
lens: fix-affected-set
outcome: reported
```

Findings: none. I checked the sets for F3 and F4, which are the two findings whose premise is a set of sites. F1 is one test row, and F2 is one summary sentence.

- **F3, driver-dead doc wording.** A grep for `driver-dead`, `dead driver` and `driver strand is dead` across non-test `.go` and `.md` files found two other sites that say when the notice goes, and neither is stale:
  - `internal/dobbyengine/doc.go:127` already names the bootstrap lock and the one-minute awake-time grace.
  - `internal/battenshed/doc.go:161` is the site the reviewer rejected. It stays accurate because the grace excludes paused intervals.

  The remaining hits (`innerrun.go:335,379`, `battenshed/doc.go:90,93`, `notice.go`, `dobbynotice.go:88`, `contracts/stencils/orch/orch-template-role.md:32,51`) name the notice or its cap and make no claim about its trigger condition.
- **F4, takeover bound.** `internal/dobbyengine/doc.go:136` is the only takeover-bound statement in the grep results, and the fix covers it.

Further tool calls used (1 of 6):
- Ran one grep over non-test `.go` and `.md` files for the driver-dead wording variants. My context did not show the other places that describe the notice.
