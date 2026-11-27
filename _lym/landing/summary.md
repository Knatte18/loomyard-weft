# Plan check: refuse a new function whose first caller is on a later card

A plan could create a function on one card and give it its first production caller on a later card.
The first card then failed its gate on the test-only-code guard (`cmd/lym/testonlycode_linux_test.go`), since no `main` reached the function yet.
The plan writer's and reviewer's prompts already said to keep a new function on the card that adds its first caller,
but no machine check enforced it, so the plan passed both plan gates and the implementer had to loosen the guard to get through.

## What changes

`internal/planparser` gets a new plan-format check, `create-caller-later`, run by `ValidateFormat` and `Validate` under `language: go`.
It fires on a card that creates a production function no ref of its own can call,
when a later card that can call the function names its handle among its targets or `Uses`.
The finding sits on the creating card and names the later card.
Its way forward: move the `Create` to that later card, or list the caller on the creating card.

A card can call the function when a ref of its `Create`, `Edit`, `Extends` or `Custom` groups names production Go code besides the function itself.
These refs do not count:

- a handle the card creates as a `func`, `type` or `const`;
- a non-Go file;
- a `_test.go` file;
- a test member.

A package self glyph, a production `.go` file, a member glyph, a created `var` and a bare symbol each count.
A function is test code, and skipped, when any of these holds:

- its unit is `<dir>_test`;
- its name opens `Test`, `Benchmark`, `Fuzz` or `Example`;
- its declaration head names a `testing.` type;
- it is `main` or `init`.

`loom-plan-checks-spec.md` documents the check, and `loom-plan-spec.md`'s card-granularity rule names it.

## Design decisions

- **A plan check, not a deferred guard.**
  The board entry offered three routes.
  Deferring the guard at the card gate would make `gatecli` learn one `cmd/lym` test and map plan handles to guard keys.
  Listing the guard file on both cards would build loosening the guard into the plan.
  A plan check catches the defect before any card runs, and the guard stays exact.
- **The check fires only when a later card can be the caller.**
  A card that creates a function and nothing else can also be a test-helper card, since a helper in a package-internal `_test.go` file is declared on the package directory.
  Requiring a later card that names the function and changes production code keeps such a card silent.
  A function no later card names is left to the guard and the plan reviewer.
- **Uncertain refs count as able to call.**
  A package self glyph, an extensionless file, a bare symbol or a referenced handle the card does not create could each hold the call.
  Counting them keeps the check from refusing a valid plan.
  The cost is a missed finding on a card whose only production edit could not hold the call, such as a constant file.

## Limits

- The check reads plan text alone, so it cannot see whether a ref the creating card edits calls the function.
  An `Edit` of any production member silences it.
- A function a later card calls without naming it in its targets or `Uses` raises nothing.

## Merge danger

**Door:** two-way
**Blast radius:** planparser
