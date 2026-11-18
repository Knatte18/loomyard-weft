## Goal

```markdown
## What was seen

The shrink study (2026-11-18) found features in `landingshed` that nobody has checked for use.
Some code may be reachable from no `main` at all, kept alive only by its own tests.

## Scope

1. Run `golang.org/x/tools/cmd/deadcode` over every `main` package in the repository: `cmd/lym` and the programs under `tools/`.
   Run it without and with `-test`, so the report separates code nothing reaches from code only tests reach.
2. Delete each unreachable function, and with it the tests that cover only that function.
   Code only tests reach is deleted the same way, unless it is a test helper.
3. Check each hit before deleting it: a function reached through reflection, a registry, a `go:linkname` or a build tag for another platform stays,
   and the report names it with the reason.
4. No behaviour change: the gate stays green, and nothing a user can call is removed.

The report lists the removed lines per package and every hit kept with its reason.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
