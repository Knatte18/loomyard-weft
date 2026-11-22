# A hardener round never approves with its focus undriven; the arbiter or the parent rules

`harden-approve-needs-focus` · Note · bug, burler, arbiter, shed

## Seen

`harden-fabric-single-repo-verbs` round 3 (sonnet, medium, 4 min) returned APPROVED with these focus entries undriven, by its own Coverage section:
the discard half of a killed teardown, the keep-local tag push, a failed step with no kill, and `lym adopt --archives` over real owner tags.
The approval settled the harden segment at once: Harden-Burler-Fix only ran its gates and the run went on to landing.
Nothing asked the arbiter or the parent, and once settled the segment cannot be reopened (`lym loom goto` refuses a review or fix row of a settled segment).
The untested cases became note `harden-fabric-undriven-cases`.

## Wanted

Operator, 2026-11-22: a hardener round never approves unless everything is in place.

- A round whose review leaves a focus entry undriven, or marks one WARN or not driven, does not settle as approved.
- The arbiter rules first: order another round over the undriven entries, or accept with a stated reason per entry.
- When the arbiter cannot rule, the run stops and asks the parent, which orders another round or accepts.

## Where

- The hardener recipe `contracts/recipes/hardener-recipe.yaml` and its Bouncer row (`harden_round`).
- The settle rule of a review segment for a reviewer verdict of APPROVED (`internal/burlerengine`, `internal/shedadapters/bouncer.go`).
- The arbiter's scope of halts it may settle (`internal/shedarbiter`, `arbiterstop`).

Related: task `code-bouncer-asks-parent`, which makes a code review ask the parent instead of settling past an open question, in the same packages.
