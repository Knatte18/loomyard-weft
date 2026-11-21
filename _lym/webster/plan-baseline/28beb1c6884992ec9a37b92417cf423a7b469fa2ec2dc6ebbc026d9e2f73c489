# Card 5 — one-repository-builder

**Create:**
- `internal/hubforge#OneRepoOptions`
- `internal/hubforge#CopyCodeBare`
- `internal/hubforge#NewOneRepoHub`
- `internal/fabricengine_test#TestCloneSingleRepo_SeedsTheNamespace`
- `internal/fabricengine_test#TestCloneOneURL_JoinsTheNamespacedHub`
- `internal/fabricengine_test#TestCloneSingleRepo_FailedSeedPushLeavesNothing`

**Edit:**
- `internal/hubforge#cloneHub` -> `func cloneHub(tb testing.TB, anchor, warpBare, weftBare, container string, singleRepo bool) (*Hub, string)`
- `internal/hubforge#NewHub`
- `internal/hubforge#CloneSecond`
- `internal/hubforge#buildTemplateIn`
- `internal/hubforge#Hub`

**Extends:**
- `internal/hubforge/doc.go`

**Uses:**
- `internal/hubforge#buildBareTemplate`
- `internal/hubforge#RegisterTeardown`
- `internal/fabriccli#CloneAndWire`
- `internal/fabricengine#CloneHub`
- `internal/fabricengine#SetSeedPushHookForTest`
- `internal/fabricengine#AbandonSeededClone`
- `internal/fabricengine#PrimeRecordsBranchKey`
- `internal/fabricengine#NewBolt`
- `internal/fabricengine#Bolt.Push`
- `internal/fabricengine#Bolt.HasUnpushed`

**Intent:**
Add `hubforge`'s one-repository builder (D8) and the clone's tests over it.

`CopyCodeBare(tb, defaultBranch)` copies the cached code bare of `buildBareTemplate` into the test's temp dir and returns its path;
for a non-empty `defaultBranch` it renames `main` there and points the bare's `HEAD` at it.
It hands a test an un-cloned origin it can seed, hook or clone itself.
`OneRepoOptions` holds `Anchor` ("." or "backend") and `DefaultBranch` (empty keeps the template's `main`).
`NewOneRepoHub` takes a bare from `CopyCodeBare` and seeds the hub with `cloneHub(..., singleRepo true)`,
which drives `fabriccli.CloneAndWire` with `CloneOptions{WeftURL: bare, SingleRepo: true, Subpath, Shortname: TestShortname}`, the code path of `lym fabric clone --single-repo <url>`.
The returned `Hub` has `CodeBare` and `RecordsBare` both set to the one bare; it registers `RegisterTeardown` as `NewHub` does.
`Hub`'s `CodeBare` and `RecordsBare` field docs say so: in a one-repository hub both name the one origin.
`cloneHub` passes `WarpURL` only when `weftBare` differs from `warpBare`, so `CloneSecond` of a one-repository hub is a plain one-URL clone, the second machine's join (D8);
`NewHub` and `buildTemplateIn` pass `singleRepo false` and keep their two-URL clone.
`doc.go` gains the copy helper, the builder and the one-URL second clone.

Tests, all tier1 in `fabricengine_test`, over `NewOneRepoHub` unless a row says otherwise:
- `TestCloneSingleRepo_SeedsTheNamespace` (D1, D3), over `CopyCodeBare` then `CloneAndWire` with `SingleRepo`:
  every `refs/heads/` ref of the bare is read before the clone and is at the same tip after it;
  the bare holds `refs/lym/heads/main` and refs/lym/heads/main-weft;
  the hub has the records sibling, `_lym` junctions and `_bolt` on `main`, `.lym-anchor` at the records root, and the board's `.lym-warp` names the bare;
  the records store's config holds `lym.primeRecordsBranch` = main-weft and its refs hold no `refs/remotes/origin/` name that is a code branch;
  `_bolt`'s `main` has `branch.main.merge` = `refs/lym/heads/main`, and `NewBolt(BoardDir).HasUnpushed` answers false;
  a default `git clone` of the bare fetches no ref under `refs/lym/`.
  A `CopyCodeBare(t, "master")` row puts the board at `refs/lym/heads/main` and the records branch at refs/lym/heads/master-weft.
- `TestCloneOneURL_JoinsTheNamespacedHub` (D3), rows:
  - `CloneSecond` of a seeded hub builds the same layout on a second machine, and its `_bolt` holds the first hub's board files and commit;
  - a `DefaultBranch: "master"` hub's second clone checks out the board on `main` with that content and records master-weft in `lym.primeRecordsBranch`;
  - a one-URL `CloneHub` with a `--shortname` other than `TestShortname` is refused naming the recorded shortname, and the container holds no hub directory;
  - after the first hub commits a `.lym-warp` naming another URL on its board and pushes it, a one-URL `CloneHub` is refused naming the binding mismatch, with no hub directory left;
  - over a `CloneSecond` hub, `CloneHub` with `Reset` and `--subpath backend` against the recorded anchor `.` is refused naming the recorded anchor, and the existing hub is kept;
    over a bare holding the board but no refs/lym/heads/main-weft, `CloneHub` with `Reset` and a `--subpath` naming a missing directory is refused before the existing hub is touched.
- `TestCloneSingleRepo_FailedSeedPushLeavesNothing` (D3) drives `fabriccli.CloneAndWire` with `SingleRepo` over a bare from `CopyCodeBare`, rows:
  - refused board: a `pre-receive` hook in the bare refuses `refs/lym/heads/main` only; the records push lands, the board push is refused,
    the clone fails naming the refusal's way forward, the container holds no hub directory, the bare holds no `refs/lym/heads/main`, and refs/lym/heads/main-weft is absent again, the restore having deleted it;
    with the hook removed, a rerun with the same `--subpath` succeeds;
  - leftover, same subpath: a bare holding refs/lym/heads/main-weft but no board, as a failed attempt leaves it, is replaced by a rerun with the same `--subpath`, and the new tip carries a fresh anchor-marker commit;
  - leftover, another subpath: the same leftover is replaced by a rerun under `--subpath backend`, and the anchor marker on the new tip records `backend`;
  - records lease lost: `SetSeedPushHookForTest`, on the records ref, creates refs/lym/heads/main-weft on the bare; the clone fails naming the same clone again,
    leaves no hub directory, writes no `refs/lym/heads/main`, and the records branch stays where the hook put it;
  - board lease lost: the hook, on the board ref, creates `refs/lym/heads/main` on the bare; the clone fails naming the one-URL clone of the other hub, leaves no hub directory,
    and the bare's refs/lym/heads/main-weft is back at the observed tip: absent in one row, and at a pre-seeded leftover tip in another;
  - restore fails: the hook, on the board ref, creates `refs/lym/heads/main` and moves refs/lym/heads/main-weft to another commit;
    the error names both the lost board lease and the failed restore, no hub directory is left, and the records branch stays at the hook's commit;
  - transient refusal: the lock file `refs/lym/heads/main.lock` is created in the bare before the clone; the board push is rejected with `reference already exists`,
    the clone fails naming the same clone again rather than the refused-host way forward, no hub directory is left, and refs/lym/heads/main-weft is back at its observed absence;
  - abandoned seed: after `CloneHub` with `SingleRepo` over a `CopyCodeBare` bare, `AbandonSeededClone(res, rec, cause)` leaves no hub directory,
    returns an error naming the same clone again, and the bare holds no `refs/lym/` ref;
  - abandoned two-URL clone: for a two-URL `CloneHub` result, `AbandonSeededClone` returns `cause` unchanged and keeps the hub;
  - unseeded board: a hub built with `CloneHub` and `SingleRepo` alone, with `PushCloneBoard` skipped, has `NewBolt(res.BoardDir).Push` refused naming the same clone again with `--reset`,
    the bare holds no `refs/lym/heads/main`, and no failed-push record is written.
  The hook rows are process-global through the seam, so that test is not parallel.

**ImpactSummary:** `cloneHub` gains a flag; its three callers are in hubforge and keep their two-URL clone, `CloneSecond` changes only for a hub whose two bares are one path, which only the new builder makes, and `Hub` changes only in its field docs.
