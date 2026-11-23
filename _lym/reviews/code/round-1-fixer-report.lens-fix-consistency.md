```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: findings below, from the change set plus one grep and one read.

1. **MEDIUM — twin of F4 left stale.** `internal/loomcli/rownotice.go:49` says "A re-call after an escalation's answer or a resumed halt is a new visit, since it spawns a fresh agent." The fixer rewrote the same claim in `contracts/orchrole/orch-role.md:296-297`, which now says a resumed halt whose agent ended before its work did resumes that agent's session. The grep for the original phrase only matched the stencil, so this paraphrase was missed. The comment now states something the code does not hold. It can be fixed by saying "a new visit" without the "fresh agent" reason. The sibling at `internal/shedrun/rownotice.go:17` ("counts higher too") stays true and needs no change.

2. **LOW — change description's test list is stale.** The `## Tests` section of `_lym/landing/summary.md` (lines 51-55) does not mention `TestStart_FailedResumeRetriesAsAFreshSession` or `TestAttach_RecordsASupersededRunningCandidateAsInterrupted`. It also does not mention the carried-instruction row, the unwritable-record row, or the `Interrupted == false` assertion in `TestStop_RecordsTheStopBeforeRemovingTheStrand`. The F9 edit touched the same file but left this section as it was. Its line "The step-stop tests now assert that every stop they record is marked interrupted" sits beside the new assertion that a deliberate stop is not interrupted, and a reader would notice.

Everything else agrees. The `run.go` branch comment ("the next start runs a fresh session") now matches the retry fallback. The `crashresume_test.go` and `gotoresume_test.go` headers match their tests.

- `grep -rn "fresh agent|resumed halt|runs a fresh session|never came up"` over `.md` and `.go`: to find twins of the changed stencil clause and the `run.go` comment, which my context lacked.
- `sed -n 40,56p _lym/landing/summary.md`: to see the `## Tests` section next to the F9 edit, which my context held only from line 50.
