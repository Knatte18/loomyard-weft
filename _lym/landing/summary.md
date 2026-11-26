# Card gate runs the packages that depend on a card's packages

A card's gate ran only the packages its plan card named.
A card whose intent reached further than its edit list could break tests in packages it did not name, and the card gate still passed.
The breakage showed up only at the batch gate or later.
Two plan rules that led to such cards change as well.

## What changes

- **Card scope widens to reverse dependents.**
  Under the `go` scoper, a `--card` scope now holds the cards' packages and, per module, every package whose build or tests reach one of them, test imports included.
  The new `impactset.DependentPackages` builds this set over the same module graph and rules the diff scope uses: the `tier0`–`tier3` tags, and the `cmd/lym` → `internal/testkit/lymbin` rule.
  `computeScope` calls it through `cardDependents`.
  A package a card has yet to create stays in the scope.
  When `go list` cannot list the module, the gate logs the cause and the scope holds the cards' packages alone.
  The listed tiers' own compiles report a module that does not build.
  The vet step, the watched-test skip and the guard step all work from the widened set.
  This covers the `card` and `batch` keys (both use `scope: card`) and `--map --card`.
  The `dotnet` and `none` scopers are unchanged.
- **New function rides with its first caller.**
  The plan-cards block, the plan spec's granularity section and the Plan-Review granularity bullet now say a new function goes on the card that adds its first production caller.
  Until now an ordinary helper-then-caller split failed the tier1 guard that every production function must be reachable from a `main`.
  Getting past it took an allowlist entry and a `Loosened-Check:` trailer, and a later card had to remember to remove the entry.
- **Repository-wide intent names its search.**
  A card whose intent covers every instance of one kind across the repository lists every file holding one, and its intent names the search that found them.
  Plan-Review now has a bullet that flags such a card when it names no search.
  The reviewer runs the card's search, or its own, and names each instance outside the card's targets.

Docs updated with the code: the `gatecli` and `impactset` package docs, the `--card` flag help, the `gate.yaml` template comment, the plan spec's verify model, the implementer stencil and `docs/overview.md`.

## Design decisions

- The issue offered two ways to find the extra packages: packages whose tests depend on the changed packages, or tests that hold the changed strings.
  The change takes the dependency route alone, which reuses the diff scope's graph and needs no text scan.
  The packages the issue reports, which assert on messages a changed package produces, depend on that package.
- For the helper-then-caller split the issue also offered two ways: keep the function and its caller on one card, or let the gate tolerate a caller a later card declares.
  The change takes the plan rule.
  It leaves the reachability guard as strict as it was.
- A failed dependents listing narrows to the cards' packages instead of failing the gate or running whole.
  This matches how the card scope already handles an unreadable diff for watched tests.

## Tests

- `TestDependentsFor` (tier0) pins the pure core over the shared hand-built graph:
  transitive and test-import dependents sorted after the seeds, a package nothing reaches, a seed the module does not hold yet, and the `cmd/lym` binary-test rule.
- `TestGateTest_Scopes` (tier1) rows for `--card` now expect `./user`, which imports the card's `./lib`, beside `./lib`.
  This covers the tier run, the vet run, the timeout row, the `--gate card` row and the `--main-packages` row.
  On the merge base those rows fail, since the scope there held `./lib` alone.
- `TestDependentPackages` and `TestCardDependents` (tier1) drive the `go list` path in-process over real temp modules.
  They cover an importer and a tier-tagged test importer being added, and the failure path.
  On failure `DependentPackages` returns an error naming the directory, and `cardDependents` keeps the cards' packages alone.

## Merge danger

**Door:** two-way
**Blast radius:** gate
