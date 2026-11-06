# Revise tier 1: what makes it slow, and which tests to change, fold or cut

`tier1-test-revision` · Task · research, fabric · running · Worktree-Create

## Decision (operator, 2026-11-06)

Tier 2 stays in the `darn-write` and `code-burler-fix` gate keys.
Tier 1 is what is slow, the `internal/fabricengine` suite above all, and the tests are due a revision.

## Wanted

A report at `docs/research/speed/tier1-test-revision-2026-11-06/report.md` that says what to change in tier 1, ranked by the time it takes off a gate pass.
It changes no code, test or config, and files no task.

Build on `docs/research/speed/fabric-test-speed-survey-2026-11-03.md` and `docs/research/speed/run-slowness-causes-2026-11-06/report.md`, and do not repeat their measuring where it still holds.

Answer:

1. Where a tier 1 pass goes now: seconds per package, the packages that set its wall, and how much is test time against binary start, build and vet.
   `lym gate report` and the gate's logs hold real passes; prefer them to new runs.
2. `internal/fabricengine`: which of the survey's levers still stand at today's tree, what each is worth now, and the order to take them in.
   Say what the pair templates cost to build, since the survey gives only what they save.
3. The same for the other slow packages: their costliest tests and fixtures, and what makes them slow.
4. Which tests can go or fold: overlapping tests, tests that pin nothing another test does not, and tests of tier 1 that need no real git or could sit in tier 0.
   Use `cmd/testtiming`'s redundancy report and read the costliest tests, and name what each candidate alone pins, so a fold keeps it.
5. Which tests can share one repository instead of each building its own, across the whole of tier 1, not only the costliest tests.
   Weigh three shapes: refusal and read-only rows on one shared hub, one longer flow test in place of several tests that build the same start, and a template to copy where a test needs a broken start state.
   For each, weigh the git processes saved against what is lost: a first failure hides the later steps, a failure may come from an earlier step, and a long serial test can come to set the package's wall.
6. Which reads in production code still start git through `gitexec` where `dotgit` or `gitrepo`'s go-git reads could answer them, with a count per call.
   Writes and remote work stay on real git, for the reasons in `internal/gitrepo/doc.go`.
7. What the whole would give: a darn run's gate before and after, alone and beside two other runs.

End with follow-up tasks, ranked, each with its expected saving, its size and its risk to coverage.

## Bounds

At most three whole-repository tier 1 runs and two tier 1 runs of `internal/fabricengine`, all through `lym gate test`.
No run on the other machine; state what carries over to it by reasoning, as the survey did.
