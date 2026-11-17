# Review rounds converge: fixer rebuttals, incremental review, stalemate

Burler review rounds circled in three ways: a fixer's rebuttal never reached the next reviewer,
a later round re-reviewed the whole target and found new nits on code it had passed,
and nothing stopped a segment whose fixes changed nothing.
This change closes all three for every segment the Bouncer drives (Discussion-Review, Plan-Review, Code-Review), on Claude and Codex reviewers alike.
Finalize's fix reviews in `internal/landingshed`, the `circling` cause and its LLM judge, and the bounce budget are untouched.

## What changes

- **Fixer-report dispositions.**
  The fixer-report gains YAML frontmatter with one `dispositions` entry per finding the fixer owns:
  `fixed`, `fixed-differently`, `disputed`, `deferred` or `ruling`.
  A `disputed` entry carries `ground`, `rebuttal` and `evidence` as `<path>:<line>` locations.
  `internal/burlerengine` is the sole parser; a new fix check joins the fix half's gate and passes on its attempt cap.
  Go-written reports (no fixer spawned) carry the frontmatter too, plus the Go-only key `spawned: false`.
- **Prior-findings block.**
  When round N-1 is complete, Go renders every finding of that round into round N's `prior_rounds`,
  each with its disposition and rebuttal beside it, plus every `Added after Discussion` entry added since round N-1 began.
  The reviewer answers each through a new `prior` list in the review frontmatter (`resolved`, `open`, `outdated`, `withdrawn`).
  An `open` finding is carried into `findings` through a new `carries` key, so the fixer's flow is unchanged.
  A `decision`-class finding held by the parent is `resolved` only by quoting the title of a listed added entry; otherwise it is `open` and carried.
- **Incremental bound.**
  In a round after the first (never an amendment generation), a new finding must be `BLOCKING` and sit on a file the round's changes file touches.
  A round-check gate entry in `internal/burlerengine/verdict.go` enforces this and the `prior`/`carries` rules, and passes on its attempt cap.
  `origin: parent` findings are exempt only in a round holding a parent-review ask, and only when their `parent_ids` name that ask's report.
  The stencils state the hunk rule and the rule that a fix's regression in an untouched file is located at the changed line that causes it.
- **Per-round snapshot.**
  The review producer records a snapshot of the review target once per round through `internal/state`,
  and writes the changes file from the previous snapshot to now.
  Source scope uses a new general fabric verb pair (working-tree id, diff of two tree ids) in `internal/fabricengine` and `internal/fabriccli`;
  overlay scope hashes the target files and keeps a copy under a `.lym` path named by `lymdirs`.
  A missing cache leaves the round without a changes file and never fails it.
- **Stalemate escalation.**
  `shedadapters.EscalationCause` gains `stalemate`.
  After the judge returns a non-converging verdict, the Bouncer counts rounds in a row whose post-fix target id equals the round's start id with a fixer run,
  and escalates once the count reaches the loom key `review_stalemate_rounds` (default 2, 0 turns it off).
  It sits after the soft-cap auto-accept, and yields to `softCapRule` where that rule would accept, using `designWrongKeys`.
  `lym loom circling accept` and `continue` settle it, and `internal/arbiterstop` settles it as it settles `circling`.
  It is Awaiting and never counts against the bounce budget.
- **`lym burler validate-review`** runs the round checks on a review in a run directory,
  resolving the worktree through `lymcwd` from the seam cwd and rebuilding the context through an exported `internal/shedadapters` builder.
  Outside a run directory, or when the worktree does not resolve, it runs the shape checks alone.
  Its help text no longer says the verb needs no git repository.
- The decision escalation's parent notice now names `lym loom decision add`, one entry per answered finding.

## Design decisions to know

- A round whose snapshot record lacks `decisions_routed`, or whose fixer-report fails the fix check, leaves the next round full with today's prior-rounds text.
  A run that crosses this binary change reviews and bounces as before until it holds one complete new-format round.
- Because each gate entry passes on its attempt cap, a review that passes on the cap keeps its out-of-bound findings and the fixer works them.
- The `ruling` disposition is the fixer's claim, never the parent's record; only an added decision entry answers a held finding.
- `review_stalemate_rounds` is `CategoryNone` in loom's `ConfigSpec`: it moves an escalation earlier or later and lets no gate pass.
- The snapshot digest keys on each added entry's heading and body, so a `lym loom decision replace` fold followed by an add is still told apart.

## Check by hand

- The `resolved` status of a held `decision`-class finding is checked only to quote a listed added-entry title, not to settle that finding;
  a reviewer can resolve one by quoting an unrelated listed entry (`internal/burlerengine/roundcheck.go`).
- A `carries` link can keep a finding open past D4's bound: confirm a carried finding cannot pass off a new defect on an unchanged file,
  in the location-path rules of `internal/burlerengine/roundcheck.go`.
- The fact that a round routes decisions is written by the fix producer into the round's snapshot record:
  confirm a segment whose fix row only notifies the parent marks the held finding in the next round's block (`internal/shedadapters`).
- Confirm the stalemate check yields to `softCapRule` in a window-open round with no design-wrong entry,
  and does not stalemate a round the soft-cap auto-accept takes (`internal/shedadapters/bouncerstalemate.go`).
- Confirm `lym burler validate-review` resolves the worktree from the seam cwd and not from the review file's path,
  since `_lym` is a link out of the worktree (`internal/burlercli/validate.go`).
