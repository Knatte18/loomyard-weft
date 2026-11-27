```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposal that the fix carried out for F12.

**1. LOW: the proposal removes the rule for new .NET-building tests**

- **Changed text:** `pattern/PATTERN-test-speed.md`, the bullet "A check that need not stop a task names `upkeep`".
- **Problem:** "a test that builds a real .NET project" was replaced by the names of three current tests. The bullet now gives a future .NET-building test no place.
- **Case:** an author adds another test that runs `dotnet build` in `internal/gatecli`. The old text told them to mark it `//lym:only upkeep`. The new list only says where three existing tests are. For a non-`cmd/lym` package, the neighbouring bullet ("a guard of any package but `cmd/lym`, stays at the task gates") then points toward the task gates if the test is a guard.
- **Staleness:** the list also goes stale when a test is added or renamed (`scribe:prose`, "Don't pin a perishable specific").
- **Change that keeps both:** state the category and cite the tests as examples, such as "a test that builds a real .NET project, such as …".

**2. LOW: the rewritten summary bullet reverses its reasoning**

- **Changed text:** `_lym/landing/summary.md`, the watched-guard design decision.
- **Problem:** it now reads "stays at the task gates, so it is no whole-repository check". Staying at the task gates is not why it isn't a whole-repository check.
- **Evidence:** the previous text had it the right way round: "stays at the task gates: it reads a bounded set of files …, so it is no whole-repository check". Reading a bounded set of files is the reason, and staying is the result.
- **Change:** "A guard that reads a bounded set of files is no whole-repository check, so a watched one stays at the task gates: …".

**The rest of the proposal held up against these cases:**

- **Upkeep run:** the watch marker does not filter the test out of the `upkeep` run. `markedScope` takes `request.marked` as its guards, and `scanGuards` and its watch drop never run for them.
- **Upkeep tiers:** the test is tier 1, like the other upkeep tests, so it adds no new refusal.
- **Docs:** the search found no other pattern, contract or package-doc claim that C# fixtures compile at task gates.
- **Accepted trade-off:** a task that adds an invalid C# fixture now lands, and the next upkeep run reports it. The summary's "a .NET break can wait a few landings" already accepts this.

**Tool calls**

- One `grep` over `pattern/`, `PATTERN.md`, `contracts/` and `internal/` docs for C# fixture compile claims. My context had only searched for the test's name and a few fixed phrases. It also named a `CLAUDE.md` that does not exist.
