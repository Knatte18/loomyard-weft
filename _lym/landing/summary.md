# Red-first gate passes a hub whose scoper is not go

A `bug`-labelled run in a hub whose test scoper is not go, such as a C# hub, no longer halts at the red-first gate.
Before, `lym gate red-first` exited `gateslot.RedFirstUnjudgedExit` there, because it can only run the diff's tests on the merge base through the Go coverage maps.
The gate held the writer's handoff until the parent recorded a `No test seam` decision by hand, in every run.
Now the gate passes that exit, since no check can show the test red on such a hub.

Closes #192

## Design decisions

- The pass sits in the gate, not the verb.
  `internal/loomshed/redfirstgate.go` treats `gateslot.RedFirstUnjudgedExit` as a pass, and the `lym gate red-first` verb still exits 5 and says no way forward is needed.
- Exit 5 passes only when the run did not time out and the test runner did not crash.
  A runner crash that ends in exit 5 still fails terminally, and a test pins that case.
- A diff that adds or changes no test (`gateslot.RedFirstNoTestExit`) still holds the run without spending an attempt, as before.
  With exit 5 no longer a hold, the hold's one-line message and the final-evaluation failure text name only that case.
- A hub that runs its own tests on the merge base is not added.
  The issue allowed skipping the check or running it through the hub's own test command, and this change skips it.
- The `No test seam` and `No bug fixed` decisions keep their meaning for a hub with a go scoper.
  The Darn-Write stencil, the darn recipe comment, `docs/overview.md` and the package docs now say a non-go hub passes.

## To check by hand

- The red-first comment in `contracts/recipes/darn-recipe.yaml` once said a check that cannot judge the tests holds the run.
  It now says a non-go hub passes the gate; confirm no other wording still claims the old hold.
- No run was made against a real non-go hub.
  The behaviour is covered by gate unit tests that stub the check with exit 5, and by the `lym gate red-first` integration test for a hub whose scoper is none.

## Merge danger

**Door:** two-way
**Blast radius:** loomshed
