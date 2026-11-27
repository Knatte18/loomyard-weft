# Card gate runs the packages that depend on a card's packages

A card's gate ran only the packages its plan card named.
A card whose intent reached further than its edit list could break tests in packages it did not name,
and the card gate still passed.
This widens the card scope and changes two plan rules, one for such cards
and one for a card split that could not pass its gate.

Closes #263
Closes #262

## What changes

- **Card scope widens to reverse dependents.**
  Under the `go` scoper, a `--card` scope holds the cards' packages and, per module,
  every package whose build or tests reach one of them, test imports included.
  The new `impactset.DependentPackages` builds this set over the diff scope's module graph and rules:
  the `tier0`–`tier3` tags, and the `cmd/lym` → `internal/testkit/lymbin` rule.
  A package a card has yet to create stays in the scope.
  The listed tier runs, the vet step, the watched-test skip and the guard step all work from the widened set.
  This reaches every gate key with `scope: card` and `--map --card`.
  The `dotnet` and `none` scopers are unchanged.
- **The guard step keeps the guards the listed runs skip.**
  `cmd/lym` depends on much of the module, so the widening puts it in many card scopes.
  Under `--card`, a `//lym:guard` test of a package inside the scope still runs in the guard step
  unless a listed tier is or covers its tier.
- **New function rides with its first caller.**
  The plan-cards block, the plan spec and the Plan-Review rubric now say a new function goes on the card
  that adds its first production caller.
  A card adding it alone fails the guard that every production function is reachable from a `main`.
- **Repository-wide intent names its search.**
  A card whose intent covers every instance of one kind across the repository lists every file holding one,
  and its intent names the search that found them.
  Plan-Review flags a card that names no search, and each instance a search finds outside the card's targets.
- **Implementer stencil.**
  The sanctioned out-of-target fix now covers a `//lym:guard` test that fails in a listed tier's run,
  as well as one that fails in the guard step.

Package docs, `--card` and `--guards` help, the `gate.yaml` template comment, the plan spec and `docs/overview.md`
follow the code.

## Design decisions

- Extra packages are found by dependency, not by searching tests for changed strings.
  This reuses the diff scope's graph and needs no text scan.
  A test that depends on a changed string without reaching the package through imports is still missed at the card.
- For the helper-then-caller split, the change takes the plan rule
  rather than letting the gate tolerate a caller that a later card declares.
  The reachability guard stays as strict as it was.
- When `go list` cannot list a module, the scope keeps the cards' packages alone instead of failing the gate
  or running whole.
  The call names the module and the cause on stderr and in the log.
  `listPackages` runs without `-e`, so one package that fails to load drops every dependent of that module.
- A computed or `--base` scope still leaves every guard test of an impacted package to the listed runs,
  so a guard of a tier they do not run is not run there.
  This predates the change; `internal/gatecli/doc.go` now states it as a bound, and it is not fixed here.
  It conflicts with the guard-step rule of `PATTERN-test-speed`.

## To check by hand

- A failed dependents listing narrowing the card gate without telling the caller,
  in `internal/gatecli/scope.go` (`cardDependents`).
  The diff now prints the narrowing on stderr and the call can still pass;
  check that this is enough.
- The implementer stencil's guard deviation not covering a guard that fails in a listed tier's run,
  in `contracts/stencils/webster/webster-body-implementer.md`.
  The diff rewords the deviation;
  check that it matches the runs of a card gate that can fail outside the card's targets.
- Not run: a card whose targets are non-Go files that a package embeds or reads.
  Whether `planparser.CardPackages` seeds the same packages the diff scope would is untraced.
- Not fully run: whether every added package can be handed to each listed tier command and vet run,
  such as one whose files are all tier-tagged, a test-only one or a platform-constrained one.

## Merge danger

**Door:** two-way
**Blast radius:** gate
