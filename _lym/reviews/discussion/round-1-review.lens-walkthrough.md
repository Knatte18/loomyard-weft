```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran one concrete case through each decision's mechanism and checked the end state against the decision's purpose. 7 of 10 decisions meet their purpose; D1 breaks for webster on a reftable repository, D4's criterion contradicts its own rule, and D7 holds as stated.

**List: each decision, the case used, purpose met or not**
- **D1, met for `List`, broken for webster:** a hub on the files backend, with the prime as main, a pair as a linked worktree and `List(l.AnchorPath())` from a pair. `dotgit.Read` gives the common dir, then `worktrees/&lt;id&gt;/gitdir` and `HEAD`, then the loose ref or `packed-refs`, then `WorktreeEntry`. This matches what `parseWorktreePorcelain` produces. On a reftable repository, `List` falls back to porcelain, but webster's `otherWorktrees` has no fallback.
- **D2, met:** a registration whose build is `newOriginHub`'s steps. The first caller runs `templateFor` up to and including `cloneHub`, then the build gets the wrapped TB. Then the template is packed and fingerprinted. `cloneHub` and `AddPair` register no `Cleanup` or `TempDir`; only `NewHub`/`RegisterTeardown` do (hub.go:258-282, 378-381). The build's `gitkit.Git` and `CurrentBranch` touch nothing the wrapped TB refuses, and the board stays at its upstream.
- **D3, met:** a copy of a template whose builder ran `publishBoard`. `publishBoard` commits `board.json` and pushes it (originreconcileended_integration_test.go:58-64), so the board's HEAD equals its upstream. `relocateHub` copies the packs untouched because of the NUL check, then rewrites `.lym-warp`. `commit --amend -- .lym-warp` and `push --force` then fold the binding into the tip. The copy is clean, and the tracking ref's reflog is written.
- **D4, purpose met, criterion contradicted:** I walked `initRepo` and `initBareRemote` with one config write after `init`. The bare initialisers write three keys today, not five.
- **D5, met:**
  - `newRecordsFixture` (replay_integration_test.go:35-40) moves from `NewHub(t, ".")` to `CopyHub(Shape{Anchor:"."})`, which gives the same anchor and no pairs.
  - In the hook, `${R%/*}` and `${R##*/}` on `/a/b` give `/a` and `b`. On `/x` they give `//x-weft`, the same path as `/x-weft`.
  - In the wrapper, `${0%/*}` on a slash-free `$0` returns `$0` itself, so the stated fallback to `.` is needed and covers it.
- **D6, met as far as I could walk it:** the `TestAnchorChange` refusal rows on one `CopyHub`. I did not read `livestate_matrix_test.go` to walk the `runCell` move.
- **D7, met:** `endedHub` (originreconcileended_integration_test.go:87-99). It runs `newOriginHub` and `originPair`, then `writeStatus` and `publishBoard`, which can run in a build. `originBoard` commits into `RecordsBare` with `originCommitFile`. Each test picks its own slug and passes it to `originPair`, so the pair stays in the test as D7 says. `newAnchorHub` (anchorchange_integration_test.go:32-38) is `CopyHub` plus two commits on the prime warp, which can be built.
- **D8, met:** the two named tests stay in the files their D7 tasks edit.
- **D9, met:** I checked which task edits which file. `endedHub` lives in `originreconcileended_integration_test.go`, not `replay_integration_test.go`, so D5 alone edits the replay file. Tasks 6 and 7 both edit `anchorchange_integration_test.go`, and 7 waits for 6.
- **D10, met:** `--all` and `--tier` are valid flags of `lym gate test` (its `--help`).

**Findings**

1. **MEDIUM, design, D1 (decision-record.md:35, 47, 211):** the reftable fallback covers `fabricengine.List` alone, so webster's sibling read fails on a reftable repository.
   - Running the case: a repository with `extensions.refStorage=reftable` reaches `otherWorktrees` (internal/websterengine/gitwrap.go:324-347). That function now reads through the `dotgit` reader, which refuses with a named error. Nothing falls back, so the call that `git worktree list` answers today now errors.
   - The Bound says "`List` falls back to the porcelain path, which stays for that case alone".
   - The criteria test the fallback only through `List`.
   - Evidence: decision-record.md:35, 47 and 211; gitwrap.go:328.

2. **LOW, consistency, D4 (decision-record.md:97-98, 220):** the rule says every initialiser writes five keys at today's values, but the bare initialisers write three.
   - D4 names `user.name`, `user.email`, `core.fsmonitor`, `maintenance.auto` and `gc.auto` for every initialiser, and says "the keys and values stay as today".
   - `initBareRemote` (internal/gitkit/gitkit.go:149-156) and `initBareRepo` (internal/hubforge/hub.go:467-476) write only `core.fsmonitor`, `maintenance.auto` and `gc.auto`.
   - So "a repository from each initialiser reads the five keys" fails for a bare repository, or forces new keys onto it, which contradicts "stay as today".
   - Evidence: decision-record.md:97, 98 and 220; gitkit.go:154-156; hub.go:474-476.

**Tool calls**
- Grepped `hubforge` for `Cleanup`, `TempDir`, `Setenv` and `RegisterTeardown`, and for the fixture definitions. I needed to know whether a base build already ties state to a TB, and where `endedHub`, `newAnchorHub` and `newOriginHub` live; the manifest had neither.
- Read the anchor, origin, ended and records fixture bodies to walk D5 and D7 on real build steps; the manifest named the fixtures but not their code.
- Read `publishBoard`, `originBoard` and `writeStatus` to see whether `endedHub`'s steps leave the hub, which is one hop past the fixtures.
- Ran `lym gate test --help` and grepped the PATTERN files for `dotgit`, to check D10's command flags and `dotgit`'s leaf entry; the manifest lacked both.
