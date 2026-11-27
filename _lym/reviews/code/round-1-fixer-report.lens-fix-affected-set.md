```yaml
lens: fix-affected-set
outcome: reported
```

Task: reran the set searches for F1, F3 and F8 and checked the other set-shaped findings against context.

**Unchecked findings:** none. F2, F4, F5, F6, F9 and F10 were checked from context without a search. F7 is not a set.

**Finding**

- **LOW.** Changed behaviour: F3, where every verify site now stops on `Result.Unjudged()` and weaver, webster, landing and loom start no fixer on a memory pressure stop.
  - The pattern rule that governs this set was not updated.
  - `pattern/PATTERN-gate-slot.md:154` still reads "Clearing a failed gate with a cold agent: only `gatefixer`, one fixer per failed evaluation; … a runner crash gets none."
  - It names only the crash, although a pressure stop now gets no fixer either.
  - If the run may not edit PATTERN without a declared change, the fixer should defer this and say so.

**Coverage otherwise holds**

- **F3:** every non-test `verifytree.Verify` consumer (`loomshed` verify and red-first, `landingshed`, `weaverengine` batch and plan verify, `webstercli`, `websterengine`) now keys on `Unjudged()`. The remaining `.RunnerCrash` readers only log it or are `verifytree` itself.
- **F1:** every production guard goes through `newMemoryGuard`, so each has `own` set. The one test literal without `own` returns no pressure limit and never calls it.
- **F8:** the only sites that set the mark are the two in `retryTier`, matching the finding.

**Outside this lens:** `internal/adoptcli` files fix tasks from gate runs and reads neither the crash line nor the pressure line. It did not read the crash line before this change either.

**Tool calls**

1. `git grep` for readers of the crash flag and the two lines: my context only had the pre-edit search.
2. `git grep` for prose about unjudged runs in markdown, YAML and the consumer packages: the earlier search skipped `pattern/` and config files.
3. `git grep` for guard constructors and `MemoryPressure`/record mark sites: needed the post-edit set.
4. `git grep` for `verifytree.Verify` and gate-line consumers: no earlier search listed all consumers.
5. `sed`/`git grep` on `pattern/PATTERN-gate-slot.md` and webster prose: that file was not in my context.
