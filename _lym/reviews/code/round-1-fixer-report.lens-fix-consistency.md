```yaml
lens: fix-consistency
outcome: reported
```

Findings (lens fix-consistency):

1. LOW — `internal/dobbyengine/doc.go:115` (the "Bound" bullet), and the `Land` field comment at `internal/dobbyengine/state.go:40`.
   - The bullet reads "a request it refuses is not asked for again in the same episode". F1 now has dobby ask `Land` again on every tick after a refusal, until it succeeds or the reading stops passing.
   - The bullet is only correct if "request" means Beam-Await's refusal of a land request, and a reader is likely to take it the other way.
   - The `Land` field comment says the record holds "a land request asked for or a notice posted". A `Refused` record is neither closed nor a finished step, so it should say the step stays open.
   - The new `Refused` doc on `LandState` is accurate, and the `doc.go` bullet added at line 114 is accurate.
   - Suggested fix: reword the bullet to "a request Beam-Await refuses", and add "or a refusal posted, which leaves the step open" to the `Land` comment.

2. No findings on the rest of the change set:
   - The `beamcli` `land.go` seam doc and `markReady` doc match the new idempotent commit, and the `deps.go` dep behaves as they say.
   - I found no stale prose on "already flagged" or "nothing is written" in a `.go` or `.md` file outside the tests. The grep returned only unrelated modules.
   - The `loomcli/doc.go` re-check paragraph now agrees with `pawlRecheckLines` and `pawlRecheck`.
   - The `tick` comment now says "the beam's landing step", which matches `landBeam`.
   - The dobbyengine test whose expectation changed (`h.lands` after a refusal) is updated, and the new test pins the open episode.

Tool calls:
1. Ran one grep over `internal` and `docs` for stale "already flagged", "nothing is written" and "ready record" wording in non-test files. My context held only the edited files, not the other docs.
