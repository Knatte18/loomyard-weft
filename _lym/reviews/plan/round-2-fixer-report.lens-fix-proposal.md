```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the R1, R2 and R3 proposals the fixer carried out, by building a case where each one is wrong or breaks something elsewhere in the plan.

**LOW**: R2's split leaves nothing stopping a whole-package `tier3` run from moving a window.
- **Changed text:** card 8, "Its doc comment states that it moves a throwaway window, so the operator runs or approves it by name", and the line saying neither test has a skip or a flag.
- **Case:** an operator runs `lym gate test --tier 3 ./internal/kwinplace` with no `-run`, as `docs/benchmarks/running-tests.md:34` allows. That run starts `TestContract_TheRealKWinAppliesAsRecorded` and moves a throwaway window on the live desktop. Before the fix, the skip kept the apply step out of such a run.
- **Why only LOW:** `tier3` is run by hand, and running it is the operator's call. That matches the review's reasoning ("`tier3` is already run by hand, one named test at a time"). Still, the card's "by name" is a convention the operator keeps, not something the test enforces.

R1 holds up:
- `Entry.Output` has no reader anywhere in the plan, so storing a different output under `off` than after an apply breaks nothing.
- Under `off`, a reopened homed window that lands off its home keeps its home and updates `LastDesktop`. The next report then does not read it as a hand move.
- The empty-landed rule only fires for a window that sits on several desktops or on all.

R3 holds up:
- `InstallNetworkGate` (`internal/tendercli/gate.go:54-58`) sets the mark only for commands that carry `clihelp.RunProcessAnnotation`. Those are shed's `run` and `step` and one beam command. `lym reed window` and `lym machine start` don't carry it, so a hand run stays unmarked.
- A placement child started from a marked process inherits the mark, so `heldByPause` holds its network calls under a pause. The child makes only local `qdbus6`, `journalctl` and `kscreen-doctor` calls, so a pause changes nothing for it.
- `t.Setenv` in `TestPlaceWindowChild` is compatible with the test, since it already avoids `t.Parallel`.

Tool calls:
1. Read `internal/tendercli/gate.go:40-125` and grepped the users of `RunProcessAnnotation`, to check which commands set the mark and what the mark changes in a child that inherits it.
2. Grepped the plan for readers of `Entry.Output` and `docs/benchmarks/running-tests.md` for the `--tier` syntax, to test R1's output split and R2's command.
