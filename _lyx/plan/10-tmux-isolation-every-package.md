# Card 10 — tmux-isolation-every-package

**Edit:**
- `cmd/lyx#tmuxIsolationFailures`
- `cmd/lyx#allowedNoTmuxMain`
- `cmd/lyx#TestTmuxIsolation_TaggedPackagesRunThroughTmuxkitMain`
- `cmd/lyx#TestTmuxIsolation_FixtureTrees`
- `pattern/PATTERN-test-isolation.md`

**Intent:**
`cmd/lyx`'s tmux-isolation scan requires the entry point for every test package, not only those with an `integration`-, `tmux`- or `llm`-tagged test file (D4).
`tmuxIsolationFailures` drops its "skip a package with no tagged file" filter: every package directory with a test file must have a `TestMain` that compiles under every tag set and on every platform that compiles any of its test files, and every `TestMain` in it must call `tmuxkit.Main`.
It still counts tagged packages, so the guard's tagged floor stays meaningful.
The failure for a package with no `TestMain` names that package and the way forward the guard already prints.

`allowedNoTmuxMain` stays empty, and its comment states what an entry is: it names one package with a reason naming why that package's `TestMain` cannot call `tmuxkit.Main`, that `TestMain` removes the process run directory after `m.Run` through `lyxtemp.Remove`, and what the entry lets through is that package's tmux isolation and its `t.TempDir` trees left in the bare system temp directory after a kill; "does not start tmux" is still no reason.
The guard's doc comment and the file's header say the rule now covers every test package, at the cost of one run directory per test binary.

`TestTmuxIsolation_FixtureTrees` keeps its rows; the "untagged package" row, a package with one untagged test file and no `TestMain`, now expects its one failure per tag set instead of none, which is the "a test package with no `TestMain` of its own is refused" criterion, and a new row, an untagged package whose `TestMain` calls the kit, expects none.

`pattern/PATTERN-test-isolation.md`'s "tmux isolation" section says every test package runs its tests through `tmuxkit.Main`, so its `t.TempDir` trees live in the process run directory; it drops the bullet keying the rule on "has a tagged test file" and the bound that a package with only untagged files never calls `Main`, and states the allowlist entry's content and bound as above.
Cards 7, 8 and 9 gave every current test package its entry point, so the guard passes on the tree.

**ImpactSummary:** The tmux-isolation guard now covers every test package; its fixture expectations for an untagged package change.
