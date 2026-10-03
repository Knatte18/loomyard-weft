---
round: 1
focus:
  - "Cards 3–10 against D2 and D4: grep the tree for every `*_enforcement_test.go`, every `go env GOMOD` exec, every `exec.LookPath(\"go\")` root-lookup skip and every `runtime.Caller` root lookup in a test that enforces a CONSTRAINTS.md invariant; settle whether each one is a target of some conversion card or sits on the `root-lookups-left-alone` list, and whether that list's three files really enforce no invariant."
  - "Card 12 against D5's deletion list: grep every `internal/*cli` package for `NoArgs`, `UnknownSubcommand`, `EveryVerbHasNonEmptyShort`, `EveryCommandHasShort` and exact-verb-set tests; settle whether any package holding such a copy (for example `configcli` or `stencilcli`) is missing from the card's Edit list."
  - "Cards 19, 22, 25, 31, 32 and 33 each promise edits beyond their listed targets (a repo-wide `format: 5` grep, a repo-wide stencil-seeding grep, 'a raw wording check in another test file', tests fixed to set `TMUX`, environment-from-scratch spawns 'found'); settle whether each target list covers what the Intent commits to, and whether the ImpactSummaries of 31 and 32 ('no test body changes') still hold."
  - "Cards 31 and 32 against D18: settle whether together they cover every package with an `integration`- or `smoke`-tagged test file under every tag set that compiles its tests, whether any edited `TestMain` file is itself `smoke`-tagged despite 'changes no smoke-tagged file', and whether each newly created `TestMain` in a git-spawning package keeps or adds `gitkit.HermeticGitEnv()` per the Hermetic Git constraint."
  - "Attack-surface question on cards 24, 25, 27 and 28: each deletes or drops assertions (generic-wording claims, `loomcli` duplicates of `shedverbs`, the duplicated commit-status tests); settle whether each card's own Intent states what the removal now lets through and what bounds it, or only defers to another card or to D12/D14/D15."
  - "D23 docs fidelity across cards 1, 2, 17, 20, 28 and 30: settle whether each new kit (`plankit`, `stencilkit`, `tmuxkit`) and `statuscommit` has its package doc carried by its card, whether `internal/testkit/doc.go` describes the kits as D23 requires, and whether card 5's claim that CONSTRAINTS.md needs no edit is consistent with D20's Tier Purity and Hermetic Env lines."
  - "Card 20's import-cycle note against cards 21–22: check `internal/stencilstore`'s and `contracts/stencils`' import graphs and settle whether any package card 21 or 22 converts (for example `friction` or `pattern`) is imported by them, which would make its internal tests unable to import `stencilkit`."
  - "Card 15 against D9: D9 says only the concrete-type column stays pinned, while the card keeps a backing-engine-name column for `coverage_guard_test.go`; settle whether the decision record licenses that column."
---

Generation 0: no rework round exists under `_lyx/loom/`, so the decision record alone is the answer key.
No card is `Custom`, so that rubric item has nothing to check.
The plan-level `## verify:` runs the untagged and `integration` tiers over `./...` and vets `smoke`, so verify coverage is not in focus.
