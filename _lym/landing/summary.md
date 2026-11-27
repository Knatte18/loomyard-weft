# Run slow whole-repository tests at landing or upkeep, not in task gates

`cmd/lym`'s test binary reaches about 2 GB, mostly from tests that read the whole repository's source,
and it is the binary the memory guard kills first.
Nearly every change puts `cmd/lym` in the impacted set, so every task gate paid for those tests.
Tests that build a real .NET project added to the gate time too.

A new test marker, `//lym:only <key>...`, limits a test to the `--gate` calls whose key it names.
With it, the slow tests move to one of two places:

- **Landing (`parent-align finalize`)**: every `cmd/lym` whole-repository guard that has no `//lym:watch`,
  except the dead-code guard, plus the tree-sitter link check and the Linux cross-compile.
  Every other key skips them, `webster` and `adopt` included, although both run the whole tree.
- **Upkeep (`upkeep`)**: the dead-code guard and the tests that build a real .NET project.
  The latter include the watched guard `TestCSharpFixtures_CompileWithDotnet`,
  so a `*.cs` change no longer compiles the fixtures at its task gate.
  No task gate and no landing gate runs any upkeep test.

The other watched `cmd/lym` guards and the guards of other packages stay at the task gates.
`TestAgentMessaging_NoEngineTool` and `TestRemovedVerbs_NamedNowhere` gain `//lym:guard` along with the landing marker.

## The upkeep run

`gate.gates` gains the key `upkeep` with a new scope, `marked`, which runs only the tests marked for the key.
The prime's board-tidy agent, on the main branch, runs it every `beam.board_tidy_every` landings.
It files a board note per dead function, and a `bug` note for any other failing test or for a refused call.
A failure that an open note already names gets no second note,
and a call refused because no test is marked for it gets none.
A beam's board-tidy agent runs no upkeep, and neither does a hub whose scoper is not `go`.

## Plan check removed

The `create-caller-later` plan check is gone, with its spec text and the stencil rules that went with it.
It existed because a card adding a function that only a later card calls failed its own gate on the dead-code guard.
No card gate runs that guard now.

## Design decisions

- Markers apply only under the `go` scoper and only to `--gate` calls.
  A call without `--gate` holds no marked test back, though it runs only the tiers it names, `tier0` by default.
  A call whose words after `--` carry a test filter holds none back either, and runs what the filter selects.
  A tier whose command holds no `{args}` holds nothing back in its listed run.
- The landing marker names both `parent-align` and `finalize`, so Finalize can carry Parent-Align's pass over an unchanged tree.
- A green record entry stores the tests it held back.
  It covers a later request only if that request holds them back too.
- A `marked` key refuses a call that would leave a marked test out because no listed tier is or covers its tier.
  In a hub that declares tiers, it also refuses a call that would run no test.
- A marker naming a key that `gate.gates` does not hold refuses the gate.
- A computed or `--base` scope now also runs a guard test of an impacted package when no listed tier is or covers its tier.
  Before, no run of such a call included it.
- Accepted costs: a break that only a landing test sees first fails a task at Parent-Align.
  Dead code, and broken behaviour that only the .NET tests see, pass every task and landing gate
  and sit on `main` until the next upkeep run.

## To check by hand

- **This hub's `upkeep` key needs setting.**
  The template gives `{tiers: [0], scope: marked}`, but the marked upkeep tests are `tier1`,
  and this hub's `tier0` does not cover `tier1`.
  Until `gate.gates.upkeep` is set to `{tiers: [0, 1], scope: marked}`, the upkeep call is refused and checks nothing.
  A binary without this change does not accept the scope `marked`,
  so update every binary that reads the hub's gate config before writing it.
- After that, run `lym gate test --gate upkeep` on the main branch and confirm that it runs the dead-code and .NET tests.
- Confirm that `beam.board_tidy_every` is not 0, since the upkeep tests otherwise never run on their own.

## Merge danger

**Door:** two-way
**Blast radius:** gate
