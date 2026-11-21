# Card 4 — clone-one-repository

**Create:**
- `internal/fabricengine#legacyBoltBranch`
- `internal/fabricengine#originLayout`
- `internal/fabricengine#readOriginLayout`
- `internal/fabricengine#refuseOneURLClone`
- `internal/fabricengine#probeCodeOriginRoot`
- `internal/fabricengine#initRecordsStore`
- `internal/fabricengine#cloneSeed`
- `internal/fabricengine#PushCloneBoard`
- `internal/fabricengine#AbandonSeededClone`
- `internal/fabricengine#pushCloneSeed`
- `internal/fabricengine#seedPushHook`
- `internal/fabricengine#SetSeedPushHookForTest`
- `internal/fabricengine_test#TestCloneHub_OneURLRefusals`

**Edit:**
- `internal/fabricengine#CloneOptions`
- `internal/fabricengine#CloneResult`
- `internal/fabricengine#CloneHub`
- `internal/fabricengine/boardweft.go`
- `internal/fabricengine#checkSubpathBeforeReset` -> `func checkSubpathBeforeReset(cwd, weftURL, warpURL, requestedAnchor string, ns gitrepo.RefNamespace) error`
- `internal/fabricengine#recordedAnchorOnWeft` -> `func recordedAnchorOnWeft(weftURL, weftStage, warpBranch string, ns gitrepo.RefNamespace) (data []byte, source string, found bool, err error)`
- `internal/fabricengine#fetchWeftBranchTip` -> `func fetchWeftBranchTip(weftURL, dir, branch string, ns gitrepo.RefNamespace) (bool, error)`
- `internal/fabricengine#probeMainWeftHasPath`
- `internal/fabricengine_test#TestCloneHub_AnchorAdoptPath`
- `internal/fabriccli#Command`
- `internal/fabriccli#runCloneWithReset` -> `func runCloneWithReset(ctx context.Context, out io.Writer, args []string, reset bool, subpath string, forceBootstrap bool, into string, shortname string, singleRepo bool) int`
- `internal/fabriccli#CloneAndWire`
- `internal/fabriccli_test#TestCommand_RefusesBeforeAnyHubWork`

**Extends:**
- `internal/fabricengine/doc.go`
- `internal/fabriccli/doc.go`

**Uses:**
- `internal/fabricengine#oneRepositoryNamespace`
- `internal/fabricengine#OneRepositoryBoardBranch`
- `internal/fabricengine#PrimeRecordsBranchKey`
- `internal/fabricengine#RecordsNamespaceRoot`
- `internal/fabricengine#RecordsNamespaceHeads`
- `internal/fabricengine#RecordsNamespaceTags`
- `internal/fabricengine#teardownHub`
- `internal/fabricengine#checkPathRequest`
- `internal/fabricengine#ownedFreshlyCreatedPath`
- `internal/fabricengine#dirtinessNA`
- `internal/fabricengine#suffixWeftPrimaryBranch`
- `internal/fabricengine#resolveEffectiveWarpURL`
- `internal/fabricengine#resolveEffectiveShortname`
- `internal/fabricengine#probeTreeHasPath`
- `internal/fabricengine#wrapProbeError`
- `internal/fabricengine#Bolt.PushRecorded`
- `internal/fabricengine_test#makeBareRemote`
- `internal/fabricengine_test#TestCloneHub_UnreachableWeftIsHardError`

**Intent:**
Teach `CloneHub` D3's one-repository clones, add the clone's board-push verb `PushCloneBoard` that carries the `--single-repo` seed push,
and wire `--single-repo` and the verb into `lym fabric clone`, so each new function lands with its first caller.

`CloneOptions` gains `SingleRepo bool`.
`CloneResult` gains an unexported `seed *cloneSeed`, set only by a `--single-repo` clone: the mark `PushCloneBoard` and `AbandonSeededClone` read, so `fabriccli` never reads the layout.
`cloneSeed` holds what the push and its teardown need: the container `cwd`, the hub path and the `createdToken` of the hub directory,
the records store and board paths, the prime's records branch, and the tip of `refs/lym/heads/<that branch>` the precondition check observed ("" when absent).

**Offline refusals**, before the network and anything is created:
`SingleRepo` with `WarpURL` set is refused with the way forward "give --single-repo one URL, the code repository";
`SingleRepo` with no `--shortname` is refused as a fresh bind's missing shortname is, naming `--shortname`.

**The precondition check**, run by every one-URL clone (`WarpURL` empty) before the hub-exists check, `--reset` and anything is created:
`readOriginLayout` runs one `git ls-remote --symref <url> HEAD refs/lym/heads/* refs/heads/lym-bolt` from `cwd`.
A failure of it is wrapped by `wrapProbeError`, so an unreachable origin still fails with the `probe weft <url>:` prefix and creates nothing;
`TestCloneHub_UnreachableWeftIsHardError`'s one-argument row reaches that branch and keeps passing unchanged.
`originLayout` holds origin's default branch (from the `HEAD` symref line), the board ref's tip, every namespaced head by local name, and whether `lym-bolt` exists.
`legacyBoltBranch` is the one place the old layout's name is spelled (D10), a named constant as the declared `PATTERN-code-conventions` change requires.
A two-URL clone runs no check.

`refuseOneURLClone` refuses, in this order, each message naming its way forward:
1. `lym-bolt` on origin, flag or not: f06acf39's layout; way forward: delete `lym-bolt` on origin and remove `.lym-anchor` and `_lym` from the code branches, then clone again.
2. With `--single-repo`, an origin holding the board ref `refs/lym/heads/main`: way forward: the same clone without `--single-repo`.
3. With `--single-repo`, a default branch holding `.lym-warp`: a records repository; way forward: a two-URL clone.
   It runs before the next check, so a two-repository weft whose board tracks the legacy root `.lym-anchor`, which holds both files, gets this way forward.
4. With `--single-repo`, a default branch that tracks a root `.lym-anchor` or `_lym` (`lymdirs.LymDirName`): f06acf39's layout; the way forward of 1.
`--single-repo` reads the default branch's root through `probeCodeOriginRoot`, a throwaway shallow clone under `cwd` removed before it returns, as `probeWeftBinding` makes one;
it reports `.lym-warp` (`bindingFound`) and `.lym-anchor` or `_lym` (`recordsFound`) through `probeTreeHasPath`; an unborn origin reports neither.
`probeCodeOriginRoot` lives in `recordsprobe.go`, the file `cmd/lym`'s destructive guard allows a probe's `os.RemoveAll` in.

**Path choice**, after the check passes:
- `SingleRepo`: the seed path below.
- No flag and the board ref present: the join path below.
- Otherwise today's two-repository one-argument path, reading the URL as a weft;
  a weft whose board tracks the legacy root `.lym-anchor` holds no `lym-bolt`, so it clones as today.

Both one-repository paths name the hub from the URL with `DeriveWarpName`, clone the code store from it as the prime with `cloneRepo`,
and make the records store with `initRecordsStore(url, weftPath, fetch)`: `git init`, `remote add origin <url>`, then before any fetch replace the fetch refspec with
`+refs/lym/heads/*:refs/remotes/origin/*` and `+refs/lym/tags/*:refs/tags/*`, set push refspecs `refs/heads/*:refs/lym/heads/*` and `refs/tags/*:refs/lym/tags/*`,
and set `remote.origin.tagOpt` to `--no-tags`, so tag auto-following can never bring a code tag; with `fetch` it then fetches through `gitrepo.New(dir).Fetch()`, under the remote-refs lock.
The namespace strings come from card 2's constants.
The records store therefore never fetches or holds a code ref (D1).
Then, as the two-repository path does: `suffixWeftPrimaryBranch` puts the records store on <prime-branch>-weft (adopting origin/<prime-branch>-weft when the join fetched one),
`ensureBoardWorktree(weftPath, OneRepositoryBoardBranch, boardDir)` checks out the board on `main`, never the code store's branch,
the clone sets the config key `lym.primeRecordsBranch` to <prime-branch>-weft in the records store, and the anchor, binding and shortname steps follow.
`boardweft.go` changes whole: `ensureBoardWorktree`'s parameter, its doc comment and the file header name the board branch it is handed, which is the code branch only in a two-repository hub.
- **Join** (`fetch` true): the board adopts `origin/main` (D3); `.lym-warp` and `.lym-shortname` are read from `refs/remotes/origin/main`.
  The binding goes through `resolveEffectiveWarpURL(recorded, found, opts.WeftURL)`, the clone URL as the supplied URL,
  so a binding naming another URL is refused and the hub torn down; the shortname goes through `resolveEffectiveShortname`, so a wrong `--shortname` is refused the same way.
  With `--reset` and an explicit `--subpath`, `checkSubpathBeforeReset` runs before the old hub is removed, with `oneRepositoryNamespace()`:
  `recordedAnchorOnWeft` and `fetchWeftBranchTip` then fetch refs/lym/heads/<branch>-weft, so the recorded anchor is read from the namespace.
  Under a non-identity namespace `recordedAnchorOnWeft` skips its `HEAD` and `branch --show-current` fallback and answers not found when the namespaced records branch is absent,
  since a one-repository board carries no legacy marker and origin's `HEAD` is the code default branch, which D2 bars a records-store read from;
  the subpath is then checked against the warp tree as on the create path.
  The two-repository callers pass the zero namespace; `probeMainWeftHasPath` is one.
- **Seed** (`fetch` false): nothing is fetched, so `suffixWeftPrimaryBranch` creates the prime's records branch fresh and `bornWeftPrimaryBranch` gives it a first commit;
  the board is orphan-created on `main`; the anchor marker is committed through the create path, whatever a leftover namespaced records branch records and whatever `--subpath` names;
  the binding is the one URL; no commit is written to any code branch.
  The result carries `seed`. Its observed tip is looked up in `originLayout`'s namespaced heads under `RecordsBranchName` of the prime's checked-out branch, read from the code clone, never under origin's `HEAD`.

**`PushCloneBoard(res, rec)`** is the clone's board push.
Without `res.seed` it is today's `NewBolt(res.BoardDir).PushRecorded(SyncOptions{}, rec)`, which a two-repository clone and a join take.
With it, `pushCloneSeed` pushes from the records store, under `oneRepositoryNamespace()`:
1. `seedPushHook(ref)`, then the prime's records branch with `UpdateRemoteBranchLeased(origin, branch, tip, observedTip, ns)`, leased on the observed tip or its absence;
2. `seedPushHook(ref)`, then the board's `main` with `UpdateRemoteBranchLeased(origin, main, boardHead, "", ns)`, leased on absence, the layout's detection key, last.
Once the board push lands, `pushCloneSeed` sets the board branch's upstream in the records store, `branch.main.remote` = `origin` and `branch.main.merge` = `refs/lym/heads/main`,
the two keys a two-repository clone's first board push writes, so `Bolt.HasUnpushed` reads the fresh board as in sync and its later pushes are default pushes.
`seedPushHook` is nil in production; it is a test seam set through `SetSeedPushHookForTest` in `export_test.go`, which restores nil on cleanup and is process-global.
It receives the origin ref about to be pushed, so a test can move origin before either push.

`pushCloneSeed` and its restore live in `destroy.go`, the one file `cmd/lym`'s destructive guard lets call `UpdateRemoteBranchLeased` and `DeleteRemoteBranchLeased`,
and `pushCloneSeed` is an executor of the destruction gate, as `PATTERN-fabric-boundary`'s destruction chokepoint requires.
Before the records push, and before each restore, it runs the gate's four checks in order:
- containment, through `checkPathRequest` on a `pathRequest` whose container is `seed.container` and whose target is `seed.hubPath`;
- ownership: that request declares `ownedFreshlyCreatedPath(seed.hubTok)`, and the ref it pushes or restores must be the prime's records branch named at the precondition check, or the board branch,
  each leased at the tip that check observed or at the tip the seed pushed;
- dirtiness, declared `dirtinessNA` with the reason that an origin ref carries no checkout file;
- force, false.
A failed check refuses before any push.
No on-disk state reaches a failed check, since the seed's own clone minted the token and named both refs, so the checks are a fail-loud wiring guard.
Each landed push appends `KindRemoteBranchUpdated`.

A failure of either push removes the hub directory through `teardownHub(rec, seed.container, seed.hubPath, seed.hubTok, cause)`.
When the records push had landed, the teardown first returns the records branch to what the check observed, leased on the tip it pushed:
`UpdateRemoteBranchLeased` back to the observed tip, or `DeleteRemoteBranchLeased` when it was absent.
A restore that fails is named in the error beside the cause, so a records branch another push moved since stays as that push left it.
The cause names its way forward:
- origin refused the push (a `*gitexec.GitError` whose stderr holds `[remote rejected]` with a reason other than the transient ones, `(failed)`, `cannot lock ref` and `reference already exists`):
  a host that accepts refs outside `refs/heads/` and `refs/tags/`, or a two-repository hub with `lym fabric clone <records-url> <code-url>`;
- the board push lost its lease (`gitrepo.ErrLeaseRejected`): origin now holds another hub's board; `lym fabric clone <url>` joins that hub, and another origin seeds a separate one;
- any other cause, a lost records lease and a transient refusal included: run the same clone again.
  `reference already exists` is transient here: git 2.53.0 prints it, under a lease or not, when another process holds the ref's lock on origin,
  and a ref that origin advertised as present fails the lease client-side before any such line.

**`AbandonSeededClone(res, rec, cause)`** is the recovery for a `--single-repo` clone that fails after `CloneHub` and before `PushCloneBoard`:
with `res.seed` set it removes the hub through `teardownHub` and returns `cause` naming the same clone again as the way forward, so no hub is left for a reconcile to push unleased;
without it, it returns `cause` unchanged.
Origin holds nothing of the seed at that point, so the rerun starts clean.

**The CLI.**
`fabriccli`'s `Command` clone subcommand gains `--single-repo` (bool, default false) and passes it to `runCloneWithReset`, which sets `CloneOptions.SingleRepo`.
The usage string gains `[--single-repo]`, and the Long help gains a paragraph:
one URL is the code repository; `--single-repo` seeds a one-repository hub whose records and board live under origin's `refs/lym/`;
a plain one-URL clone of an origin holding that board joins it; the flag is refused for an origin that already holds one, a records repository, a second URL, and an origin left by the old `lym-bolt` layout.

`CloneAndWire` replaces its board push, `b.PushRecorded(SyncOptions{}, rec)`, with `fabricengine.PushCloneBoard(res, rec)` at the same step:
after the Bolt commit of `.lym-warp`, `.lym-shortname` and the hub-wide config, before the junction wiring.
It reads no layout; the verb pushes a two-repository clone's and a join's board as before and a `--single-repo` result's seed (D3).
Its doc comment names the step.
A seed push that fails has already removed the hub, so the error returns with nothing left to reconcile.
Every failure between `CloneHub` and `PushCloneBoard` (the hub-wide reconcile, the sparse-marker write, the Bolt commit) returns through `fabricengine.AbandonSeededClone(res, rec, err)`,
which removes a `--single-repo` hub and names the same clone again as the way forward;
a two-repository clone's failure there returns unchanged, its hub left for `lym fabric reconcile` as today.
A process ended between `CloneHub` and `PushCloneBoard` leaves an unseeded `--single-repo` hub that no returned error reaches;
`lym fabric reconcile` and every other board push there meet card 2's unseeded-board refusal, which names the same clone again with `--reset`,
so the reconcile path never pushes an unseeded board unleased or replays it onto another hub's board.

Bound: `--single-repo` reaches only an origin with no namespaced board branch, and writes only under `refs/lym/` there, through the two leased pushes and the leased restore;
it replaces a namespaced records branch only at the tip the check observed, and its board push lands only while origin holds no board.
The flag reaches only `CloneHub`'s refusals and seed path.

Tests:
- `TestCloneHub_OneURLRefusals` (tier1, `fabricengine_test`) drives `CloneHub` over bares built in the test with `gitkit`, one row per refusal:
  a bare with `lym-bolt` without and with `--single-repo`; with `--single-repo`, a bare holding `refs/lym/heads/main`, one whose default branch holds `.lym-warp`,
  one whose default branch holds both `.lym-warp` and `.lym-anchor` (asserting the two-URL way forward), one whose default branch tracks `.lym-anchor`, one tracking `_lym`,
  and a clone with no `--shortname`.
  Each row asserts the error names its way forward and that the container holds no hub directory and no probe directory.
- `TestCloneHub_AnchorAdoptPath` gains a one-URL row: a two-repository weft whose board tracks the legacy root `.lym-anchor` and holds no `lym-bolt` clones and adopts that anchor.
- `TestCommand_RefusesBeforeAnyHubWork` gains a row: `clone --single-repo <url> <url>` fails before any hub work, with an envelope naming the one-URL way forward.
The join, the seed, its failures and `AbandonSeededClone` are tested in card 5, over this card's wiring and card 5's builder.
`fabricengine`'s `doc.go` gains the one-repository clone section: the check, the two paths, the seed push and the abandon step;
`fabriccli`'s `doc.go` gains the flag and the push step.

**ImpactSummary:** `CloneHub` gains two branches reached only by a one-URL clone that the new precondition check routes there; the two-repository path is unchanged apart from three probe helpers gaining a namespace parameter whose two-repository callers pass the zero value; in fabriccli one flag and one call swap, and `CloneAndWire`'s callers (`runCloneWithReset`, `CloneAndWireFromLocalBares`, hubforge) see the same board push for every two-repository clone.
