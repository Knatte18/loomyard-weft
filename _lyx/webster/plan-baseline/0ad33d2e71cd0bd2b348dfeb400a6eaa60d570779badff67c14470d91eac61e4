---
format: 5
approved: true
language: go
---

# Plan: test-suite-weight

Make the test suite cost a fraction of what it does today, measured per tier and per package before and after, and leave nothing running behind it.
The harness lands first and records the before-state; the kits then make every tmux-tier server hermetic and non-login, build `lyx` once per gate run, run it under group kill, and fail a package that leaves a child; hub fixtures become templates, copies and a shared read-only hub, and every hub-taking test is reclassified; `lyx`'s own geometry and ref lookups move in-process; two scans pin the tiers and `PATTERN-shuttle-stop`; the last card re-measures.

## Card Index

1 — fixture-git-marker — fixture git carries a marker set by the gitkit and hubforge spawn helpers and, process-wide, during a hub build
2 — testtiming-resources — `cmd/testtiming -resources` measures one package at a time in a gate slot: wall, CPU, peak memory, process count, leftovers, load average
3 — testtiming-git-census — the harness counts git processes by subcommand through trace2, split fixture and code on the marker
4 — weight-report-before — the before-state report per tier under `docs/benchmarks/` and its regeneration command
5 — tmuxkit-hermetic-server — `Socket` and `KillOnCleanup` pre-start a hermetic non-login server, and reed's stale-holder probe recognises it
6 — tmuxkit-package-server — one server per package for the tests that name their own sessions, and every reed-booting tmux- and llm-tier test registers its key before boot
7 — tmuxkit-private-tmpdir-leftovers — `Main` owns a private `TMPDIR` and fails a package that leaves a child, with the `/proc` probes as kit exports
8 — reedcli-probes-onto-tmuxkit — reedcli's pid, argv, cwd probes and watchdog recognizer move onto the kit's
9 — loomcli-resume-split — the resume-reconcile test's arming half moves to a `tmux` file ahead of the socket check
10 — tmuxkit-socket-check — `Main` fails a package whose socket directory holds an unexpected server
11 — lyxbin-prebuilt — a prebuilt `lyx` named by an environment variable, else one build per test binary
12 — lyxbin-run — `lyxbin.Run` runs the built `lyx` under group kill and the per-package runners move onto it
13 — gatecli-prebuild — `lyx gate test` builds `lyx` once for a tagged run and strips the variable on an untagged one; the harness does the same, and a verify command never sees the variable
14 — claudeengine-prebuilt-deny — a spawned agent's command naming the variable is denied
15 — gitkit-no-detached-gc — the hermetic config disables detached gc
16 — hubforge-templates — shapes, per-binary templates, the relocating copy and the shared read-only hub with its write check
17 — reclassify-fabricengine-add-to-dot — fabricengine's hub-taking tests, first third, onto the template model
18 — reclassify-fabricengine-head-to-push — fabricengine's hub-taking tests, second third
19 — reclassify-fabricengine-reconcile-to-worktree — fabricengine's hub-taking tests, last third
20 — reclassify-fabriccli-and-geometry — fabriccli and the geometry, preflight, config, board, ide and stencil tests
21 — reclassify-loom-webster-shed — loom, landing, webster, shed and batten tests
22 — reclassify-reed-orch-shuttle — reed, orch, shuttle, seat, treadle and burler tests
23 — dotgit-leaf — the one `.git` parser, a standard-library-only leaf
24 — lyxcwd-walkup — `lyxcwd` finds the worktree root through `dotgit` and spawns no git, and the one read-only hub test moves onto the shared hub
25 — gitrepo-geometry — `gitrepo` reads git dir, common dir and toplevel through `dotgit` once per handle, with parity rows
26 — gitrepo-ref-reads — branch existence, ref SHA, HEAD's branch, a ref's tree and a branch's upstream through go-git, with parity rows
27 — gitexec-drop-gitdir-env — every git child loses `GIT_DIR` and `GIT_WORK_TREE`
28 — fabricengine-geometry-reads — fabricengine's git-dir, exclude-path and repo-exists spawns become geometry joins
29 — fabricengine-branch-and-ref-reads — fabricengine's branch-existence, HEAD-existence and ref-SHA spawns move onto gitrepo
30 — fabricengine-head-upstream-tree-reads — fabricengine's current-branch, upstream and tree spawns move onto gitrepo; the survivors are named
31 — tierpurity-tmux-constraint — the tier-purity scan bars tmuxkit spawns from files whose constraint lacks `tmux` and `llm`
32 — removestrand-scan — a scan pins `PATTERN-shuttle-stop`
33 — weight-report-after — the after-state report and the comparison

## Shared Decisions

### Decision: names-the-plan-picks

- **Decision:** the fixture git marker is `LYX_FIXTURE_GIT`, declared once as `gitkit.FixtureGitEnv`; the prebuilt-binary variable is `LYX_PREBUILT_LYX`, declared once as `gateslot.PrebuiltLyxEnv`; the leftover grace is `tmuxkit.leftoverGrace`, two seconds, a parameter of `afterRun` for the kit's own test; the shape set is `.` and `backend`, each with no pair and with the one pair `hubforge.TemplatePairSlug`, which is `pair`.
- **Rationale:** the record leaves these to the plan; one declaration site each keeps the scans and the deny from drifting.
- **Applies to:** cards 1, 7, 11, 13, 14, 16

### Decision: reed-exit-empty-probe

- **Decision:** reed's `sessionlessSocketHolderPersists` gains one early return: a socket holder whose global user option `@lyx_test_server` answers `on`, which only tmuxkit's config sets, is never stale. Bound: an operator config that sets that option disables the reap for its own servers; reed's own boots never set it.
- **Rationale:** the decision record's parent amendment "D4: reed's stale-holder probe skips the hermetic test server" licenses it. D4 pre-starts a session-less server and keeps it alive with `exit-empty off`, which tmux 3.6 honours, verified; but reed reaps any session-less holder that persists across `staleSocketGrace`, so without the probe every cold boot would wait five seconds and replace the hermetic server with a config-less one. Keying on `exit-empty off` instead would also skip the reap for an operator whose `~/.tmux.conf` sets it.
- **Applies to:** card 5

### Decision: hub-fixture-classification

- **Decision:** a hub-taking test is shared when every hub access it makes is a read (a file read, a link read or an in-process go-git read); it copies the nearest shape when it writes (a fabric mutating verb, `SeedConfig`, `AddPair`, any run of the `lyx` binary, any in-process CLI verb, any spawned git); it stays on `NewHub` when its anchor is outside the shape set. The nearest shape is the anchor the call names, with the one pair when the test's first act is `AddPair` of a slug it reads back only through the `Hub` accessors, and no pair otherwise, the test keeping its own `AddPair`. Shared: `internal/fabricengine/warplayout_test.go`, moved in card 24, since until card 24 removes `lyxcwd`'s `rev-parse --show-toplevel` spawn the test spawns git and so is not read-only. Fresh: `TestSpawnAnchoredScenario` in `internal/ideengine` and the `wts/some-task` case of `internal/fabricengine/weftgit_exclude_test.go`, plus hubforge's own tests of `NewHub`. Every other hub-taking test file, listed in cards 17 to 22, copies.
- **Rationale:** D11's rule, applied over a census of every `hubforge.NewHub` call site: all but one run a verb, seed config or spawn git, and two name an anchor outside the set.
- **Applies to:** cards 16 to 22, 24

### Decision: rev-parse-survivors

- **Decision:** after cards 28 to 30 a `rev-parse` spawn survives at `InstallPostCheckoutHook` (`--git-path hooks`, which honours `core.hooksPath`), `resolveArchiveTip` (`FETCH_HEAD^{commit}`, revision syntax over git's fetch output) and websterengine's `--disambiguate` and `<commit>:<path>` probes; `gitrepo`'s own `MergeHeadPresent` (`--verify --quiet MERGE_HEAD`) stays, since D12 moves fabricengine's spawns and lists the ref reads it moves, and the `MERGE_HEAD` pseudo-ref is none of them; `verifytree`'s `HEAD` and `HEAD^{tree}` reads and `impactset`'s `<base>^{commit}` are outside the record's scope and untouched.
- **Rationale:** the amended D12 keeps every answer git derives from config or revision syntax; everything else is a geometry or ref read gitrepo now answers.
- **Applies to:** cards 28, 29, 30

### Decision: tagged-verification-through-the-gate

- **Decision:** a card whose change is only observable in the `tmux` tier carries a `Verify:` line of the form `lyx gate test --tags tmux <packages>`, run as a background Bash call, with `-- -run '^$'` where compiling the tier is the point; the plan-level verify never runs `tmux`.
- **Rationale:** the card gate runs tier 1 and the batch gate tier 2; the kits' server, socket and runner changes would otherwise first run at Publish.
- **Applies to:** cards 5 to 10, 12, 22, 28 to 30

### Decision: test-economy-and-budget

- **Decision:** new behaviour is pinned at the kits' and packages' public surfaces, as rows or steps of an existing test where one covers the behaviour, and a new test function only for a behaviour none covers; `tmuxkit`'s post-run check is tested through `afterRun` and `checkSockets` in-package because `Main` runs outside any test. A card whose new functions push a package past its budget row lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:` and raises the row in the same commit.
- **Rationale:** `PATTERN-test-economy` and the test budget.
- **Applies to:** all cards

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
