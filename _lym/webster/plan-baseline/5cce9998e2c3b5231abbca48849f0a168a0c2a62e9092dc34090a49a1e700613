# Card 5 — one-repository-clone

**Create:**
- `plan:internal/fabricengine#PrimeBranchConfigKey` -> `const PrimeBranchConfigKey = "lym.primeBranch"`
- `plan:internal/fabricengine#cloneOneRepository` -> `func cloneOneRepository(rec *Mutations, cwd, hubPath string, opts CloneOptions, origin cloneOrigin) (CloneResult, error)`
- `plan:internal/fabricengine#cloneOrigin` -> `type cloneOrigin struct`
- `plan:internal/fabricengine#readCloneOrigin` -> `func readCloneOrigin(primePath string) (cloneOrigin, error)`
- `plan:internal/fabricengine#PushOneRepositoryClone` -> `func PushOneRepositoryClone(res CloneResult) error`
- `plan:internal/hubforge#NewOneRepoHub` -> `func NewOneRepoHub(tb testing.TB, anchor string) *Hub`
- `plan:internal/fabriccli_test#TestCloneAndWire_OneRepository` -> `func TestCloneAndWire_OneRepository(t *testing.T)`
- `plan:internal/hubforge#TestNewOneRepoHub` -> `func TestNewOneRepoHub(t *testing.T)`
- `plan:internal/boardengine_test#TestBoardWrite_OneRepositoryHub` -> `func TestBoardWrite_OneRepositoryHub(t *testing.T)`

**Edit:**
- `internal/fabricengine#CloneOptions`
- `internal/fabricengine#CloneResult`
- `internal/fabricengine#CloneHub`
- `internal/fabricengine#ensureBoardWorktree`
- `internal/fabricengine#commitWeftAt`
- `internal/fabricengine#WireJunctionsWith`
- `internal/fabricengine#wireJunctionSet`
- `internal/fabriccli#Command`
- `internal/fabriccli#runCloneWithReset` -> `func runCloneWithReset(ctx context.Context, out io.Writer, args []string, reset bool, subpath string, forceBootstrap bool, into string, shortname string, singleRepo bool) int`
- `internal/fabriccli#CloneAndWire`
- `internal/hubforge#cloneHub` -> `func cloneHub(tb testing.TB, anchor, warpBare, weftBare, container string, singleRepo bool) (*Hub, string)`
- `internal/hubforge#NewHub`
- `internal/hubforge#CloneSecond`
- `internal/hubforge#buildTemplateIn`
- `internal/hubforge#SeedConfig`
- `internal/hubforge#ReanchorPair`
- `internal/fabriccli_test#TestCommand_CloneEngineRefusalReachesEnvelopeUnchanged`

**Extends:**
- `internal/fabricengine/doc.go`
- `internal/fabriccli/doc.go`
- `internal/hubforge/doc.go`

**Uses:**
- `plan:internal/fabricengine#ProtectedBranchRefusal`
- `internal/fabricengine#oneRepositoryAt`
- `internal/fabricengine#BoltBranchName`
- `internal/fabricengine#teardownHub`
- `internal/fabricengine#commitAnchorMarker`
- `internal/fabricengine#WriteShortname`
- `internal/fabricengine#scratchLinkTarget`

**Intent:** D2, D11 and the wiring half of D3: `lym fabric clone` takes the one-repository layout, the Bolt sits on orphan `lym-bolt`, and a one-repository worktree is wired with no junction.

Clone (`CloneHub`, new field `CloneOptions.SingleRepo`, flag `--single-repo` registered in `Command` and passed through `runCloneWithReset`):
- `--single-repo` with a second URL refuses before anything is created, naming the flag; way forward: give the one repository's URL alone.
- One URL: `CloneHub` clones it as the prime (`cloneRepo`, the hub created through `createExclusiveDir` as today) and `readCloneOrigin` reads, from the clone, whether `origin/lym-bolt` (`BoltBranchName`) exists and whether origin's default branch holds the `.lym-warp` binding or `.lym-anchor` at its root.
  - Origin holds `lym-bolt`: the one-repository clone. The prime's branch must hold `.lym-anchor` at its root, else the clone refuses naming the missing marker;
    way forward: commit at the root of origin's `<branch>` a `.lym-anchor` holding the anchor `lym-bolt`'s own `.lym-anchor` names, then run the clone again.
    `ensureBoardWorktree` adds `_bolt` as a worktree on `lym-bolt`, now adopting `origin/lym-bolt` with its upstream when no local branch exists. Nothing is pushed.
  - `--single-repo`: refuses when origin holds `lym-bolt` or a weft binding, naming which; way forward: run the clone without `--single-repo`, or with it against a repository holding neither.
    Otherwise it commits `.lym-anchor` with the anchor at the root of the prime's branch (`commitAnchorMarker` on the prime),
    and creates `lym-bolt` locally as an orphan through `ensureBoardWorktree`, seeded as a fresh weft bind's Bolt is seeded today (`.lym-anchor`, `WriteShortname`, committed). `CloneHub` pushes nothing.
  - Neither `lym-bolt` nor `--single-repo`: today's meaning, the URL is a weft URL; that path is unchanged.
- Every refusal after the hub directory exists goes through `teardownHub`, so a refused clone leaves no hub directory and running it again starts clean.
  `CloneResult` gains an unexported field carrying the hub's creation token, so `PushOneRepositoryClone` can tear the hub down under the same ownership proof.
- Both one-repository paths set `lym.primeBranch` (`PrimeBranchConfigKey`) in the prime repository's git config to the prime's checked-out branch, through `gitexec` as fabric's other config writes do.
- `CloneResult.WeftBase` is the prime's anchor directory, so `CloneAndWire`'s config commit lands on the prime's branch as the seed records D2 names.
- `CloneAndWire` in that layout skips the weft-binding record and the two-repository junction steps, commits the hub-wide config on `lym-bolt` and the module configs on the prime's branch without pushing,
  then calls `PushOneRepositoryClone` before `hubreconcile.Ensure`, which pushes the board as today.
- `PushOneRepositoryClone(res)`, for a `--single-repo` clone, pushes the prime's branch with its upstream, carrying the marker and the seed records, and only after it lands pushes `lym-bolt` with its upstream.
  A refused prime push pushes nothing further, tears the hub down through `teardownHub` and returns an error naming git's push error;
  one `ProtectedBranchRefusal` matches names the protection, way forward: allow direct pushes to `<branch>` for the account lym pushes as, then run the clone again;
  any other, way forward: settle what the error names, then run the clone again. For the one-URL clone over `lym-bolt` it pushes nothing.
- Bound: `--single-repo` creates only; it cannot recognise an existing two-repository hub's code repository, whose origin holds neither branch nor binding, and run there it starts a separate one-repository hub over the same code and changes nothing in the other hub.

Wiring (`WireJunctionsWith`, `wireJunctionSet`): for a worktree `oneRepositoryAt` reads as one repository it creates `_lym` and `.lym` as real directories at the anchor,
seeds only the `/<anchor>/.lym` exclude line (through `seedGitExclude`), the heddle lines (`seedWarpExcludeLines`) and the `.scratch` link,
and skips `seedLymJunction`, `.lym` content adoption and `seedWeftArtifactExcludes`.
The exclude never gets `_lym`, `.lym-anchor` or the weft artifact lines. `scratchLinkTarget` already resolves to the worktree's own `.lym/scratch` through card 3's `RecordsWorktreePath`.
The Bolt checkout of a one-repository hub keeps its locks in its own gitdir (card 3); `CloneHub` and `commitWeftAt` skip `seedWeftArtifactExcludes` for it,
and the exclude gets `/.lym` for the Bolt checkout's root, the `.lym` entry D3 names, so `commitWeftAt`'s stage-all never sweeps hub scratch into a board commit.

Hub fixture (D12): `cloneHub` takes `singleRepo`; with an empty `weftBare` it clones the one bare by one URL, with `SingleRepo` set from the flag, and skips `forgetCloneCorrespondence`.
`NewOneRepoHub` copies the warp template bare only and builds through `cloneHub` with `singleRepo` true; the returned `Hub` has `CodeBare` set and `RecordsBare` empty.
`CloneSecond` of such a hub clones `CodeBare` by one URL with `singleRepo` false, the D2 path over an origin holding `lym-bolt`.
`NewHub` and `buildTemplateIn` pass `false` and keep today's behavior.
`SeedConfig` stages only the config files it wrote (`git add --` those paths) in place of `git add .`, so a seed in a one-repository hub never sweeps code into its commit; a two-repository seed commits the same files as today.
`ReanchorPair` fails the test through `tb.Fatalf` on a hub with an empty `RecordsBare`, naming `NewOneRepoHub`, since it moves a records sibling such a hub lacks.

Tests:
- `TestCloneAndWire_OneRepository` (tier1, package `fabriccli_test`, bares built through `gitkit`, a hook planted in the bare for the refused rows) drives `fabriccli.CloneAndWire`. Rows:
  `SingleRepoOnAFreshOrigin` (marker and seed records committed and pushed on the prime's branch, `lym-bolt` on origin, `_bolt` on `lym-bolt`, `lym.primeBranch` set, `_lym` and `.lym` real directories, `.lym` excluded, no `_lym`, `.lym-anchor`, `.weft/` or legacy lock exclude line);
  `SingleRepoRefusesAnOriginHoldingLymBolt`, `SingleRepoRefusesAWeftBinding`, `SingleRepoRefusesASecondURL`;
  `ProtectedMarkerPushLeavesNothing` (pre-receive hook printing `GH006: Protected branch update failed`: error names the protection, no `lym-bolt` on origin, no hub directory);
  `RefusedMarkerPushLeavesNothing` (hook refusing with another message: error names it, same leftovers check);
  `OneURLOverLymBolt` (a second clone of the first row's origin takes the one-repository layout with `lym.primeBranch` set);
  `OneURLOverLymBoltWithoutAMarkerRefuses` (origin with `lym-bolt` and a prime branch lacking the marker: refused naming the marker, no hub directory);
  `BoardCommitNeverStagesHubScratch` (a file under the hub scratch directory stays out of a `Bolt.Commit`).
- `TestBoardWrite_OneRepositoryHub` (tier1, package `boardengine_test`, `NewOneRepoHub`) is D11's row: a board write through `boardengine`'s sync commits and pushes on `lym-bolt`, origin's prime branch stays where it was, and no code branch changes.
- `TestNewOneRepoHub` (tier1) rows: `IsAOneRepositoryHub` (no `weftname.Suffix` sibling, `PrimeRecords` equals `PrimeWorktree`, `.lym-anchor` tracked at the root); `CloneSecondTakesTheSameLayout`;
  `SeedConfigCommitsOnlyTheConfig` (a staged code file stays staged and out of the seed's commit).
- `TestCommand_CloneEngineRefusalReachesEnvelopeUnchanged` (tier1) gains a row passing `--single-repo` with two URLs, so the flag's wiring reaches the engine's refusal.

**ImpactSummary:** Clone, its CLI flag, the Bolt commit and junction wiring gain one-repository branches; the two re-signed members have one and three callers, all listed; two-repository clone, wiring, seeding and every NewHub user keep today's behavior.
