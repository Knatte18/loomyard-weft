# Fixer report: discussion round 2

Every finding was checked against the code and fixed in `_lyx/discussion/decision-record.md`.
`lyx loom validate-discussion` passes over the edited pair.

## Fixed

### F1 (BLOCKING, design): the arm builds loom's shed

Confirmed: `loomCLI.specFor` fills `BuildShed` with `buildLoomShed` or `loomrecipe.New` (`internal/loomcli/arm.go:274-278`), and `Routing` comes from `loomrecipe.Routing` (`arm.go:185`, `reject.go:169`, `wiring.go:819`), none recipe-aware.
D1 now decides that loom's arm, shared by `Arm` and `ArmAt`, reads the seed's recipe and picks the shed builder and routing from it for every verb that needs either.
It drops the false claim that no verb reads a recipe and names the verbs that stay unchanged because they build no shed and read no routing.
The `PATTERN-driver-choice-single-site` constraint line now points at the arm's recipe read.
A new acceptance criterion pins that `lyx shed step`, `lyx loom resume` and `lyx loom goto Darn` build the darn shed over a darn seed.

### F2 (BLOCKING, design): the reader's call-time stop had no way forward

Confirmed against `pattern/PATTERN-refusal-way-forward.md` and the bare wrap in `internal/landingshed/verifygate.go:156,189`.
D6 now gives the reader's error the way forward `lyx config darn --set verify=<command>` from the prime, then `lyx loom resume`, and states that the landing wrap keeps the reader's message.
D7 now names the empty-`verify` refusal as one spec row covering start and every gate, Publish and Finalize call, rather than adding a third row.
The start check reaches the same reader, so one refusal with one way forward has one row naming each point that reaches it.
The Constraints entry and the acceptance criterion now assert that the way forward appears in the halt reason.

### F3 (MEDIUM, design): `verify_attempts` checked only at start

Confirmed against `loomrecipe.New`'s `checkReviewBudget` on every build.
D4 now has the darn builder refuse a `verify_attempts` below 1 as its first act at every build.
D7 makes that the second refusal row, with the way forward `lyx config darn --set verify_attempts=<n>`, then `lyx loom resume`.
A new acceptance criterion covers a value lowered after the start.

### F4 (MEDIUM, design): "never on a landed run" did not hold on hand removal

Confirmed: `Finalize.markTaskDone` only warns on failure (`internal/landingshed/finalize.go:246-256`), and `haltedStep` keeps batten's `Run-Shed` waiting on a halted child (`internal/battenshed/innerrun.go:621`).
D10 now treats a run as landed when the teardown's landed flag is set or when the child's own status, read before removal, holds a `Finalize` Done, and marks it done on every route.
It states that batten stays `running` while its child halts, so a spent-budget darn keeps its claim, and that the warning names `lyx batten pause` as the abandon step.
The attack-surface paragraph now names the one landed case the clear still lets through, a merged Finalize whose done mark failed and whose push returned Stuck, and bounds it.
A new acceptance criterion covers a landed but unmarked run removed by hand.

## Disputed

None.

## Deferred

None.
