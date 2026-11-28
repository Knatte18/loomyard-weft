# Reviews and plans check their claims against the code: fix keys, test infrastructure, card claims and first callers

`review-fix-claims-checked` · Task · bug, burler, stencil, webster · running · Friction-Reflect

Imported from [issue #285](https://github.com/Knatte18/loomyard/issues/285).

## What happened

In a Code-Review round, a BLOCKING finding under `PATTERN-test-stand-ins` proposed a fix that depended on test infrastructure the repository does not have.
It asked for the affected tests to run over a shared session kit under `internal/testkit/` "whichever carries the recorded session stand-in", with `reed` and `creel` real.

- No kit carries a recorded LLM session stand-in. `docs/research/no-fake-other-modules.md` says it "does not exist yet".
- The same file says `testkit/shuttlefake` is not a stand-in for an LLM session, and lists it as a standing module stand-in.
- A real `reed` needs tmux, which the affected `tier0` tests cannot spawn. The file puts a real `reed` at `tier2`.

The fixer could do only part of the finding and deferred the rest.
The partial fix itself needed new stand-ins, which the fix self-check then reported against `PATTERN-test-stand-ins`:

> fix-affected-set, MEDIUM: F2 is only partly met and adds stand-ins. Fixed differently: F2 is recorded as deferred with the remainder named above.

The deferral reason in the fixer report:

> Reed, creel and the LLM session are still stood in for: no kit holds a recorded session stand-in, a real reed needs tmux while these tests are tier0, and shuttlefake is itself a standing module stand-in. The parent decides whether to accept this or to build the infrastructure.

The judge left the finding open and tagged it `design-wrong`, and the segment stopped `awaiting` on cause `budget` with that BLOCKING finding open.

## Why it is abnormal

A proposed fix that cannot be carried out costs a fixer pass, adds code that breaks the rule the finding enforces,
and ends in a parent escalation over a choice the review could have stated when it raised the finding.

## What to do

Have the review step check that any test infrastructure a proposed fix names (a kit, a recording, a stand-in, a tier able to run it) exists.
When it does not, the finding says so and offers a fix that is possible without it,
or is raised as a `decision` finding: accept the gap, or build the infrastructure.
The review stencils are under `contracts/stencils/burler/`.

Provenance: task `pr-review-stalls-no-session`, note `burler-code-r1.md`.


## From issue #288

Imported from [issue #288](https://github.com/Knatte18/loomyard/issues/288).

## What happened

In a Code-Review round 1, two findings carried a `fix` key that could not be applied as written.
The fixer found this only by applying each one and running the gate, so each cost a failed gate run.

- One fix asked to move read-only tests from `hubforge.CopyHub` to `hubforge.SharedHub`, and named a test to start with.
  That test writes into its hub (a shared file lock can create the lock file).
  The cleanup failed with `shared fixture ... was written to`, and the poisoned template then failed later tests that copy it.
  The fixer disputed the finding as "premise false" and reverted the change.
- One fix asked a test file to use an identifier that is unexported in another Go package
  (the file belongs to the external `_test` package), so it did not compile.
  The fixer applied it differently, through a re-export.

## Why it is abnormal

A `fix` key reads as a checked instruction.
Both of these rested on a claim that reading the code could have disproved:
whether the named test writes to the hub, including through the code it calls,
and whether the identifier is visible from the file's declared package.
The review passed both on unchecked.

## What to do

Have the review stencils require that a proposed fix is checked against the facts it depends on before it is written:
the package clause and export status for any identifier the fix reuses,
and the callee chain for any claim that code only reads.
When the reviewer cannot establish such a fact, the `fix` text should say it is unverified.

Provenance: task `webster-loom-tests-parallel`, note `burler-code-r1.md`.


## From issue #291

Imported from [issue #291](https://github.com/Knatte18/loomyard/issues/291).

## What happened

Several cards of one approved plan stated things the code contradicts, and each implementer had to deviate:

- A test was to count through the remote-attempt seam of `internal/gitexec`, which other packages cannot reach.
- Some fields of the status push request were said to be read only by the push child, but `PushPairAnchored` also read one for its teardown lock and takeover read. The fix fell outside the card's targets, and a takeover subtest had to be removed.
- The board pusher was to yield to a pause through the gate, but the gate only holds a process that `cmd/lym` reads as a detached child, so `--board` had to join that read.
- Finalize's check for an owed push before the lock was scoped to the pull-request modes `trail` and `review`. `review` is the template default and can land locally, which the same card excluded from the check. Read literally, every default local landing on an owed parent would stop, breaking what the preceding card delivered.
- A test was to reach a specific halt with a non-ignored untracked file, but such a file stops the earlier clean check, and git reports the collapsed directory rather than the file.

## Why it is abnormal

The plan passed review with claims that reading the targeted code would have disproved, including one that contradicted another card.

## What to do

Have plan review verify each card's claims about readers, visibility and reachability against the code, and check cards against each other.

Provenance: task `push-retry-not-halt`, notes `05-pusher-interval.md`, `06-pusher-target-kinds.md`, `13-finalize-pr-owed-check.md`.


## From issue #290

Imported from [issue #290](https://github.com/Knatte18/loomyard/issues/290).

## What happened

A plan card added several exported functions to a new package whose only callers came in later cards.
The card gate then failed on `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`, once a later card made a main package import that package.
The implementer added temporary allowlist entries in `cmd/lym/testonlycode_linux_test.go`.
Each later card that adds a first caller then meets a stale-entry failure and has to remove its entry.
The cold fixer could not decide the failure and left it.

## Why it is abnormal

The plan's card split could not pass the repository's own reachability guard, and neither planning nor plan review noticed.
The fix falls outside the card's targets, so the implementer and fixer have no sanctioned way through.

## What to do

Have plan writing and review keep a new production function in the same gated unit as its first caller,
or give cards a defined way to carry a temporary allowlist entry and to retire it.

Provenance: task `push-retry-not-halt`, note `05-pusher-interval.md`.
