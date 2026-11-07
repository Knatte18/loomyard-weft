```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked every bound, guard and claim the record makes and report the ones that broke.

**Claims attacked**
- D1, the porcelain parity points (main path resolved, path order with ignorecase, relative `gitdir`, missing `gitdir` left out, missing `HEAD` listed detached, prunable ignoring expiry): held, on the record's scratch evidence and git's `pathsort`/`fspathcmp` behaviour.
- D1 Bound, the reftable fallback: broken for the `websterengine` read. Only `List` is given a fallback.
- D1, the bare main repository stays an error: held.
- D2, the per-key lock plus the ref-counted fixture-build mark allowing concurrent builds: held (`internal/hubforge/hub.go:44-65`).
- D2, "AddPair, AddPairWith and the gitkit commit helpers register none" of `Cleanup`, `Setenv`, `TempDir`: held. `AddPairWith` registers none (`hub.go:425-432`); only `gitkit` `CopyRepo` takes `TempDir` (`gitkit.go:341`).
- D2, the refusing `TB` guaranteeing "no build … places a path outside the template root": broken.
- D2, the duplicate-key and failed-build lookups: held as stated.
- D2, the board HEAD equal to its upstream after a build: broken by a detached push that may still be running (see finding 2).
- D3 Bound, that a stray `SharedHub` write is the whole of what the dropped per-copy walk misses: broken. A build's detached child is a second writer.
- D3, that the relocation rewrites only text and a pack or `packed-refs` holds no template path: held. `relocateHub` skips files containing NUL, and a pack is compressed.
- D3, the amend naming the binding file in place of a separate `add`: held.
- D3, "today's SharedHub sites only read": held. The one site is `internal/fabricengine/warplayout_test.go:26`.
- D4, "keys and values stay as today" against the criterion "reads the five keys" from each initialiser: broken for the bare initialisers.
- D5, the parameter expansion agreeing with `dirname`/`basename`, and the installed hub keeping its old hook: held.
- D9, that D5 alone edits `replay_integration_test.go`: held. `endedHub`, `newOriginHub` and `originPair` live in other files.
- D9, that tasks 7 and 8 edit disjoint files: held as far as the definitions show.

**Findings**

1. MEDIUM · design · D2 (`decision-record.md` "Bound" and the `testing.TB` bullet). The Bound overclaims what refusing `TB.TempDir`, `Setenv` and `Cleanup` contains.
   - Counterexample: a build function that calls `os.MkdirTemp`, `lymtemp.MkdirTemp`, `os.Setenv` or `git config --global` still escapes the root. Under `HermeticGitEnv` the global config is one shared file per binary (`internal/gitkit/hermetic.go:60`).
   - The relocation only checks for references to the template root (`internal/hubforge/template.go:320-326`, `findRootReference`). A path outside the root survives, and every copy then shares it.
   - That breaks "a test that wrote to its copy touches nothing shared", with no failure naming the key.
   - The Bound needs to state this gap and what bounds it, or the record needs a check that refuses it.

2. MEDIUM · design · D2 and D3 (the board check after a build, and D3's Bound). A build that commits through fabric spawns a detached push that can outlive the build.
   - `Commit` spawns the async push via `spawnDetachedPushFn` (`internal/fabricengine/commit.go:82,152`). D7 moves builder steps over "as they are", including fabric commits; `hubforge/seed.go:73` already uses `Bolt.Commit`.
   - The child can still be writing the template's bares, tracking refs and reflogs after the build returns. That is after D3's pack and after the fingerprint.
   - So the HEAD-equals-upstream check races the child and can fail or pass at random.
   - With the per-copy walk gone, copies taken while the child writes get a torn tree and nothing catches it.
   - D3's Bound names only `SharedHub` writes. Unverified: whether `gitkit.FixtureGitEnv`, which is set during builds, is one of the skip variables `SpawnDetachedPush` honours. If it is, this holds; the record should state that as the bound.

3. LOW · design · D1 (Bound and the `websterengine` read). The fallback is stated only for `List`.
   - D1 moves `otherWorktrees` (`internal/websterengine/gitwrap.go:323-346`) onto the same reader. The reader refuses a non-`files` ref backend outright, even though this caller needs paths only.
   - On a reftable repository the `websterengine` read now fails where today's `git worktree list` works, and the record says nothing about it.

4. LOW · consistency · D4 and its acceptance criterion ("reads the five keys at today's values"). The bare initialisers write three keys today, not five.
   - `internal/gitkit/gitkit.go:153-156` (`initBareRemote`) and `internal/hubforge/hub.go:473-476` (`initBareRepo`) write only `core.fsmonitor`, `maintenance.auto` and `gc.auto`.
   - Either the criterion fails for them, or meeting it adds `user.*` to bares, which contradicts "the keys and values stay as today".

**Tool calls**
- Grepped `internal/fabricengine` for the fixture and folded-test definitions: to check D9's file-disjointness claims; the manifest had no locations for them.
- Grepped `hubforge` and `gitkit` for `Cleanup`, `TempDir` and `Setenv`: to test D2's claim that the build helpers register none; the manifest lacked the helpers' bodies.
- Grepped `pattern/` for `dotgit`: to confirm which background holds `internal/dotgit` for D1's constraint.
- Read the head of `cloneHub` and the `AddPair`/`AddPairWith` bodies: one hop past D2's claim about which helpers register cleanups.
- Grepped for detached spawns and the `Bolt.Commit` definition: to find a process a build spawns that could outlive the build.
- Grepped `commit.go` for the detached push and read the `Bolt.Commit` lines: to confirm a fabric commit spawns the detached push.
