## Goal

```markdown
Imported from [issue #208](https://github.com/Knatte18/loomyard/issues/208).

The red-first gate runs the diff's new tests on the merge base only through the Go scoper. In a C# repository (NORCE Models, a .NET solution tested with NUnit) it cannot select or run those tests, and it halts every writer's handoff until the parent records a 'No test seam' waiver by hand. The gate never runs there, so a C# run gets no red-first protection at all.

Expected: the gate supports C# as a first-class language.
- Select the tests the diff adds or changes, by test project and fully qualified name (dotnet test --filter FullyQualifiedName~...), from the changed *.cs files under test projects.
- Build and run them on the merge base in a scratch checkout, through the hub's own C# test command or tier configuration, and judge red on the merge base and green at the head as for Go.
- Treat a test that does not compile on the merge base, because it reads a member the change adds, as red with that reason, not as a missing seam.
- Halt only when the selection or build fails for a reason the writer can act on.

Related: #192, which reports the halt itself.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
