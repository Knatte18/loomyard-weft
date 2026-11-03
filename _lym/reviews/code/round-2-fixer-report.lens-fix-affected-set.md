```yaml
lens: fix-affected-set
outcome: reported
```

Two sites still carry the old single-cause wording.

**Findings**

- **LOW** — `internal/preflightshed/doc.go:19-20`
  - Summary: the doc still describes the stencils check only as a version-skew refusal, with the way forward "a re-step with a binary that fills them".
  - Evidence: the refusal now also covers a blank value and gives "or give the blank value text" as its second way forward. This is the same set as F2, a site naming the refusal's cause and way forward, and the fix did not update it.

- **LOW** — `internal/loomcli/darnwiring.go:224`
  - Summary: the comment says the check halts "a stencil newer than the running binary" and omits the blank-value cause.
  - Evidence: it is a twin of the `doc.go` and `prompt.go` text the fix updated.

**F1 set (sites describing the Preflight stencils check's scope)**

- The `Env.PreflightStencils` doc now says Darn-Write only.
- `darnwiring.go:224`, `entries_simple.go:17` and `preflightshed/preflight.go:67` do not claim broader coverage than the Darn-Write table, so they contradict nothing.

**F2 set**

- Updated: `prompt.go` (comment and error), `table_test.go:210-211`, and `seatengine/doc.go:35`.
- The other `table_test.go` rows match shorter substrings that still hold.
- `wiring_darn_test.go` asserts only `seat "darn"`, the stencil name and the marker, so the new wording does not break it.

**Unchecked:** none.

**Calls**
- One grep over `internal` and `docs` for the old wording and the check's doc sites, to rerun the F2 and F1 set searches. My context held only the edited sites, not the other files that carry this wording.
