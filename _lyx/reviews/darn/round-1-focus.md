---
round: 1
focus:
  - "cmd/lyx/csharpfixtures_integration_test.go: does the fixture scan cover every `.cs` file under any `testdata/` directory, and do the skip list and the no-fixture failure behave as the summary claims?"
  - "The same file: is the throwaway project isolated, with no `bin/` or `obj/` left in the repository and no leftover dotnet processes, and does a compiler error fail the test while warnings pass?"
  - "The `integration` build tag and the `//lyx:guard` marker: do they match how neighbouring repository scans in `cmd/lyx` are tagged and guarded, and does PATTERN.md or any doc the change type names require an update that the diff omits?"
  - "Test economy: does any existing test already cover fixture validity, and does the diff add behavior, such as the `dotnet --version` target-framework parsing, with no coverage?"
---

Seed focus from the board entry (compile every `.cs` fixture in a throwaway project, skip without dotnet, run in a tier that .NET machines use) and the change description's claims.
