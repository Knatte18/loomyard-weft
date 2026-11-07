```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: I attacked each bound, guard, ordering promise and claim about existing code in the decision record, and found 1 BLOCKING, 5 MEDIUM and 3 LOW findings.

**Findings**

1. BLOCKING, D3: folding the copy's binding with go-git leaves the remote-tracking ref's reflog out of step with the ref.
   - Today `push --force` writes an "update by push" reflog entry for the board's `refs/remotes/origin/<branch>`.
   - go-git moves a ref without writing a reflog entry, so the copy's newest push entry names the template's pre-amend SHA while the ref names the amended one.
   - `internal/fabricengine/reflog.go` reads exactly that reflog (`reflog show --format=%H%x09%gs <trackingRef>`) to judge "what origin did to a branch this machine pushed".
   - The origin and records tests that build on copies (`newOriginHub`, D7's origin templates) could therefore see a history rewrite that never happened, or pass for the wrong reason.
   - D3 claims the copy stays "as today", but only the clean status, the one commit and the bare's tip are pinned.
   - Fix: D3 writes the reflog entries that `commit --amend` and `push` write today (HEAD, the branch and the tracking ref), and an acceptance criterion pins the tracking ref's reflog.

2. MEDIUM, D1: the rule "prunable when `gitdir` is missing, unreadable or invalid" probably does not match `git worktree list`.
   - From memory of git's `worktree.c`, `get_linked_worktree` returns NULL when `worktrees/<id>/gitdir` cannot be read (`goto done`), so `get_worktrees` leaves that entry out of `list`.
   - Those reasons appear only on the prune path (`should_prune_worktree`).
   - Counterexample: delete `.git/worktrees/x/gitdir`. git lists nothing for `x`, but the reader as specified emits an entry marked prunable.
   - The scratch experiment only covered a `gitdir` that points at a moved directory. The rule needs checking against git and the acceptance list needs a row for a missing `gitdir`.

3. MEDIUM, D1: the record says nothing about reading a worktree while `git worktree add` is still writing it.
   - From memory, `add_worktree` creates `worktrees/<id>/`, writes `locked` ("initializing"), then `gitdir`, then a placeholder HEAD, then updates HEAD.
   - Another lym process (dobby, a parallel run's `Add`) calling `List` in that window can find the directory with no HEAD file, or a zero HEAD.
   - git's list tolerates this. A reader that errors on a missing HEAD would make `List`, and every verb that calls it, fail where it succeeds today.
   - The bound should say that a half-written entry is skipped, or reported as git reports it.

4. MEDIUM, D3 bound against D6: the claim "the repository has one `hubforge.SharedHub` site, a read-only one" is true today (only `internal/fabricengine/warplayout_test.go`), but D6 breaks it.
   - D6 puts `TestAnchorChange`'s three refusal rows on a `hubforge.SharedHub`, and D3's bound rests on that one site.
   - If an anchor-change refusal path runs git (a `status` that refreshes a racily-clean index rewrites `.git/index`) or writes a log file under the hub, it writes to the template. Copies taken before that test's cleanup inherit the write, because D3 removed the per-copy walk.
   - The record does not establish that those refusal paths start no git and write nothing.

5. MEDIUM, D9: the claim "every other pair of tasks edits disjoint files" is false for tasks 5 and 6.
   - `newRecordsFixture` (task 5, D5) and `TestRecordsReplay` (task 6, D6, whose `EndedSet` rows move onto `hubforge.CopyHub`) are both in `internal/fabricengine/replay_integration_test.go`.
   - Both tasks "wait for nothing", so they conflict, and both convert `NewHub` to `CopyHub` in the same file.
   - Fix: one depends on the other, or the `EndedSet` rows move into D5.

6. MEDIUM, D2: refusing `Cleanup` from the build's `TB` can catch hubforge's own helpers.
   - `hubforge.NewHub` and `copyBares` call `tb.TempDir()`, which registers a cleanup, and `RegisterTeardown` calls `tb.Cleanup`.
   - The record says the template is "the base shape's build plus the build function", but not how the base reaches the template root.
   - If the base is built by copying it with `hubforge.CopyHub`/`hubAt` (which calls `RegisterTeardown`), or a builder calls `hubforge.NewHub` or `hubforge.CopyHub`, the refusal fails every registration.
   - D2 should say that the base is relocated into the template root as `templateFor` does today, and that only the helpers a template build already uses (`AddPair`, the commit helpers) are allowed in a builder.

7. LOW, D7 live state: `Arrange` returns a fixture value that `runCell` passes on, and D2's "build returns nothing" drops it. Each verb case needs a step that rebuilds its fixture value from the copy's `Hub`. The comment at `internal/fabricengine/livestate_matrix_test.go` that `Arrange` serves a non-hub target says some arrange state may not live in the hub at all, and a template root does not relocate that.

8. LOW, D1 order: git's `list` sorts linked worktrees with `fspathcmp`, which ignores case where `core.ignorecase` is true (Windows, macOS). A byte-order sort differs there, which matters for the parity test on the hub's other machine.

9. LOW, D3: writing the tracking ref directly skips the remote-refs lock of `PATTERN-gitrepo-client-boundary`. It is held, because today's `push --force` in `hubforge` also writes it without the lock and each copy has one owner, but D3's Constraints line could say so.

**Claims attacked**

| Claim | Result |
|---|---|
| D1 parity: order main-first and path-sorted | held on Linux; LOW 8 elsewhere |
| D1 prunable rule | broken, finding 2 |
| D1 relative `gitdir` | held |
| D1 detached and unborn | held |
| D1 bare stays an error | held |
| D1 reftable fallback | held (`websterengine` needs no refs) |
| D1 standard library only and spawns nothing | held |
| D1 a concurrent `worktree add` | broken, finding 3 |
| D2 lock per key with the ref-counted mark | held (`markFixtureBuild` is ref-counted) |
| D2 build-failure latch | held if the latch is set in a defer |
| D2 `Cleanup`/`Setenv` refusal | broken against hubforge helpers, finding 6 |
| D3 one read-only `hubforge.SharedHub` site | held today, broken by D6, finding 4 |
| D3 a pack holds no template path | held |
| D3 packing against the housekeeping tests | held: the test makes its own loose objects and checks that it has |
| D3 "as today" for the copy's board | broken by the reflog, finding 1 |
| D3 remote-refs lock | held, LOW 9 |
| D4 per-repository layer and one write | held |
| D5 parameter expansion against `dirname`/`basename` | held, including `/`, a Windows drive and `$0` with no slash |
| D5 `CopyHub` for `newRecordsFixture` | held: reflog clone messages are text and get rewritten |
| D7 the `newBadReedUpFixture` seed runs on the copy | held: the seed comes after `AddPair` in the fixture |
| D7 start state the same by construction | held except live state, LOW 7 |
| D9 disjoint files | broken, finding 5 |
| D8 | held |
| D10 | held |

**Tool calls**

- Grepped `internal/hubforge/hub.go` for `Cleanup`/`TempDir`/`Setenv` and the `newBadReedUpFixture` body order: the writer's context had neither hubforge's cleanup use nor the order of the seed against `AddPair`.
- Grepped for `hubforge.SharedHub` sites, the file holding `newRecordsFixture` and `TestRecordsReplay`, what `reflog.go` reads, and the housekeeping tests' loose-object guard: the context had the survey's dated count of `SharedHub` sites but not the current one, nor which file holds the two replay symbols, nor `reflog.go`'s inputs.
- Grepped `livestate_verbs_test.go` for `TempDir`/`Setenv`/`Cleanup` and `AddPair` for cleanups: the context had nothing on what the verb arranges and `AddPair` register.
