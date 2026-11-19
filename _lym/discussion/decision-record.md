# Decision record — one-repository hubs: records on their own refs, the layout known only to fabric

## Goal

Redesign the one-repository hub so its records and its board never touch a code branch.
A one-repository hub keeps one origin repository, and holds two local stores of it:
the code store, as today's prime, and a records store wired as a two-repository hub's weft but for its board's branch.
On origin the records store's branches and tags live under the custom namespace `refs/lym/`,
which no default clone fetches and GitHub lists as no branch.
Only fabric knows which layout a hub has; every other module sees a two-repository hub.
The redesign removes what f06acf39 built only to keep records on the code branch, and keeps its general pieces.

## Scope

In:

- The one-repository layout: one origin, a code store and a records store, records and board under origin's `refs/lym/`.
- `lym fabric clone` for that layout, by one URL, with and without `--single-repo`.
- Every remote operation of the records store mapped to the namespace: fetch, push, listing, upstream config, archive tags,
  teardown and remote cleanup.
- Removing f06acf39's records-on-code-branch machinery, inside fabric and out, and reverting its two-repository widenings.
- `hubforge`'s one-repository builder and the tests of the layout.
- `README.md`, `docs/getting-started.md`, `docs/overview.md`, `internal/fabricengine/doc.go` and the PATTERN entries f06acf39 changed.

Out:

- Converting a hub between layouts, including a hub cloned with f06acf39's layout.
- Any behaviour change to a two-repository hub beyond D7's reverts.
- A pull request or GitHub API path for records: records never take part in a pull request in either layout.

## Decisions

### D1 — One origin, two local stores, records under `refs/lym/`

A one-repository hub's local geometry is a two-repository hub's in every part but the board's branch:
the prime code worktree, the records store as the weft sibling `<name>-weft`, `_lym` junctions into it,
`_bolt` as the records store's board worktree, and `.lym-anchor` at the records root.
Both stores have the same origin URL, and the board's `.lym-warp` binding names that URL.
On origin, the records store's branches live under `refs/lym/heads/` and its tags under `refs/lym/tags/`.
The code store keeps git's default refspecs, so it never fetches or pushes a records ref.
The records store never fetches or holds a code ref: it is configured with D1's refspecs before its first fetch.
The board branch has the fixed name `main`, at `refs/lym/heads/main` on origin,
never derived from the code store's default branch, so a renamed code default branch cannot hide the board.
The board worktree's local branch is that constant too, never the code store's branch, where a two-repository hub keeps it:
over a code default branch `master`, the board is on `main`.
The prime's records branch is `<prime-branch>-weft`, at `refs/lym/heads/<prime-branch>-weft`,
named from the code store's checked-out branch at clone, as in a two-repository hub.
The clone records that name once, in the records store's git config key `lym.primeRecordsBranch` (new).
In a one-repository hub `primaryWeftBranch` reads that key instead of deriving the name from the board's branch,
so the orphan sweep and `resolveManagedBranch` never treat the prime's records branch as an orphan.
A later `lym fabric checkout` on the prime moves the code and records worktrees but never writes that key,
so it cannot move the prime's records branch.
A one-repository records store without the key fails closed, as an unreadable board branch does today,
and the refusal names its way forward: set the key in the records store to `<prime-branch>-weft` with `git config`.
In a two-repository hub `primaryWeftBranch` reads the board's branch, as today.
The namespace root `refs/lym/`, its `heads/` and `tags/` parts, the board branch name and the config key are named constants in fabric.

Rationale: records never sit on a code branch, so merges, pull requests, replay, the prime's push and code identity
need nothing layout-specific; the operator's clutter concern is met because GitHub lists only `refs/heads/` and `refs/tags/`.
Evidence, git 2.53.0 against local bare repositories:
a default `git clone` and `git fetch` brought only `refs/heads/*` and `refs/tags/*`;
a store configured with fetch `+refs/lym/heads/*:refs/remotes/origin/*` and `+refs/lym/tags/*:refs/tags/*`,
and push `refs/heads/*:refs/lym/heads/*` and `refs/tags/*:refs/lym/tags/*`,
pushed `git push origin <branch>` to `refs/lym/heads/<branch>`, fetched it back as `origin/<branch>`,
pushed lightweight and annotated tags to `refs/lym/tags/`, and honoured `--force-with-lease` through the tracking ref.
Under that push config a bare `git push origin` also pushed every local tag.
GitHub accepts pushes to refs outside `refs/heads/` and `refs/tags/` other than its read-only `refs/pull/`:
a push of `refs/notes/commits` to GitHub is reported updated in a 2023 git mailing-list thread
(lore.kernel.org/git, Brooke Kuhlmann), and GitHub's docs name `refs/pull/` as the hidden, read-only namespace.
GitHub publishes no page that lists accepted namespaces; D3 makes the clone refuse on a host that rejects the push.
Git's documentation pins the mechanics:
git-fetch(1) and git-push(1) define the `<src>:<dst>` refspec mapping that `remote.<name>.fetch` and `remote.<name>.push` hold;
git-push(1) applies `remote.<name>.push` only when the command line names no refspec, so an explicit refspec bypasses it;
and git-push(1) documents `--force-with-lease=<refname>` taking its expected value from the remote-tracking ref.
The D1 and D4 acceptance criteria pin the behaviour as tests over `hubforge`'s one-repository bare origin.

### D2 — The layout lives in the records store's remote config; one translation carries it

The records store's configured `remote.origin.fetch` refspec is the layout:
its source side under `refs/lym/heads/` means one repository, `refs/heads/` means two.
Fabric reads it per records store and caches only a read of an existing store, for the process.
A path with no store on disk reads as two repositories, logs nothing and is not cached.
A store whose remote config cannot be read is an error, never a two-repository answer.

Fabric hands every remote operation of the records store a remote ref namespace, the identity in a two-repository hub.
It covers, in both directions:

- every ref name fabric or `gitrepo` sends to the records store's origin: push destinations, lease refs, fetch sources,
  remote deletes and `ls-remote` patterns;
- `ls-remote` selection: a records-store listing selects by namespaced pattern, never by `--heads` or `--tags`;
- every ref name read back: `ls-remote` output and push result lines map back to `refs/heads/` and `refs/tags/` names;
- the upstream config fabric or `gitrepo` writes in the records store: `branch.<b>.merge` names the namespaced ref.

`gitrepo` takes the namespace as a parameter of every operation that sends or writes an origin ref name:
push, lease, push-result matching, the upstream `branch.<b>.merge` write after a push, and the remote deletes, plain and leased.
It learns no layout.
No records-store path reads origin's `refs/heads/` or `refs/tags/` in a one-repository hub,
since those hold the code repository's branches and tags.
Fabric never runs a bare `git push` in the records store.
A remote `HEAD` read for the records store answers D1's board branch name `main`, never origin's `HEAD`.
The translation never touches the code store.
No module outside fabric and `gitrepo` reads the namespace, and none reads the layout:
`lymcwd` keeps no layout predicate,
and `battencli`, `loomcli`, `statuscommit`, `landingshed` and `hubreconcile` take no layout input.
`.lym-anchor` sits only at the records root, as in a two-repository hub.

Bound: the translation acts on the records store's origin alone, so a code push can never land in the namespace;
a misread fails closed, as an error, except for a missing store, which has nothing to push.

Rationale: the refspec is what git itself obeys, so a second record of the layout could disagree with it.
A push to a URL with an explicit destination bypasses the configured push refspec,
so the namespace is passed to every operation explicitly rather than left to git's config.
The cached, quiet read carries over the requirement `two-repo-merge-probe-regression` sets for the layout probe it replaces.

### D3 — Clone

`lym fabric clone <url>` with one URL takes the one-repository layout when `git ls-remote` finds D1's board ref
`refs/lym/heads/main` on origin.
It clones the code store as the prime, makes the records store by initialising it with D1's refspecs and then fetching,
checks out the board on D1's board branch and the prime's records branch, records the prime's records branch as D1 states,
and wires the hub as a two-repository clone does in every other step.
`ensureBoardWorktree` is handed the board name, never the code store's branch,
so the board adopts `origin/main`, which D1's fetch refspec maps from `refs/lym/heads/main`.
One URL whose origin holds no namespaced board branch takes today's two-repository path, reading the URL as a weft.

`lym fabric clone --single-repo <url>` clones the code store, then makes the records store by `git init` with D1's refspecs,
never by the two-repository weft clone, and fetches nothing into it:
a fetch would bring a leftover records branch that the fresh bind's seeding then adopts,
and the seed's leases take the tips the precondition check observed.
It seeds that store as a fresh two-repository bind seeds its weft,
except that `ensureBoardWorktree` is handed the board name, so the board is orphan-created on D1's board branch,
and the board's `.lym-warp` names the one URL.
It records the prime's records branch as D1 states.
It writes no commit to any code branch.
The records store adopts nothing from origin:
it creates the prime's records branch fresh, never from `origin/<prime-branch>-weft`, and commits the anchor marker on it.
A records branch an earlier failed attempt left in the namespace without a board branch is therefore replaced,
whatever anchor it records and whatever `--subpath` the rerun names.

The seed is pushed at the step of `CloneAndWire` where a two-repository clone pushes its board:
after the Bolt commit of `.lym-warp`, `.lym-shortname` and the hub-wide config, and before the junction wiring.
That push is one fabric verb, which in a two-repository hub is today's board push.
`CloneHub` marks a `--single-repo` result, so `fabriccli` calls the verb without reading the layout,
and the verb then pushes the seed in this order:

1. the prime's records branch, holding the anchor-marker commit,
   leased on the tip, or the absence, observed at the precondition check;
2. the board branch, holding the board commit, leased on absence (`--force-with-lease=refs/lym/heads/main:`);
   it is the layout's detection key.

A seed push that fails, refused, unreachable or on a failed lease, removes the hub directory the clone made and names its cause.
When the records push had landed, the teardown first returns the records branch to the state the precondition check observed,
leased on the tip it pushed: it restores the observed tip, or deletes the branch when it was absent.
Ways forward:

- origin refused the push: a host that accepts refs outside `refs/heads/`, or a two-repository hub;
- the board push lost its lease: origin now holds another hub's board;
  a one-URL clone joins that hub, and another origin seeds a separate one;
- any other cause: the same clone again.

Every one-URL clone refuses, before creating anything.
The check runs `git ls-remote` on origin once, reading origin's `HEAD`, the board ref, the prime's records ref
`refs/lym/heads/<default-branch>-weft` and `lym-bolt`; the prime is cloned on origin's default branch, so that ref is its records branch.
It refuses:

- with `--single-repo`, an origin holding the namespaced board branch; way forward: the same clone without `--single-repo`;
- an origin whose default branch holds `.lym-warp`, which is a records repository, when `--single-repo` is given;
  way forward: a two-URL clone;
- with `--single-repo`, a second URL;
- an origin holding the branch `lym-bolt`, or, with `--single-repo`, whose default branch tracks a root `.lym-anchor` or `_lym`,
  which is f06acf39's layout;
  way forward: delete `lym-bolt` on origin and remove `.lym-anchor` and `_lym` from the code branches, then clone again.

A two-repository weft whose board still tracks the legacy root `.lym-anchor` holds no `lym-bolt` and takes no `--single-repo`,
so it clones as today.

Bound: `--single-repo` reaches only an origin with no namespaced board branch and writes only under `refs/lym/` there.
It replaces a namespaced records branch only through a push leased on the tip it observed,
its board push lands only while origin still holds no board branch,
and a clone whose board push fails leaves the records branch as it found it, unless another push moved it since.

Rationale: seeding through the existing weft bind keeps one seeding path,
its departures limited to the store's `git init`, the board's branch and the recorded prime's records branch,
and the seed push takes the two-repository board push's own step, so `fabriccli` gains no layout step.
Pushing the prime's records branch at clone is added behaviour, since a two-repository bind leaves it to the first push;
the seed pushes it first so a hub that reads as one on origin always carries its anchor.
The detection key goes last, so a failed push leaves nothing on origin that reads as a hub,
and the teardown leaves nothing on disk.

### D4 — Pair lifecycle in the namespace

Add, push, pull, replay, hand-over, teardown, archive and remote cleanup run their two-repository code paths
in a one-repository hub, through D2's namespace.
A pair's records branch `<branch>-weft` lives on origin at `refs/lym/heads/<branch>-weft`.
Its archive tag `archive/<slug>/<tip>` lives at `refs/lym/tags/archive/<slug>/<tip>`.
Teardown deletes the records branch by its full namespaced name, since a short name cannot reach `refs/lym/heads/`.
On the path `gitrepo` takes, a push to the origin URL,
`git push <url> --delete refs/lym/heads/<missing>` exits 0 printing `remote: warning: deleting a non-existent ref` and `[deleted]`
(git 2.53.0, bare local origin), so the delete's result cannot tell an absent branch from a removed one.
A full `refs/heads/` name, the identity namespace of a two-repository hub, deletes a missing ref as silently.
The plain delete therefore first runs `git ls-remote` on the full namespaced name, in both layouts,
`refs/lym/heads/<branch>-weft` or `refs/heads/<branch>-weft`;
an absent ref answers "not deleted" and pushes nothing, so the destruction gate records no removal.
The leased delete names the observed tip, so an absent branch fails its lease, as in a two-repository hub.
Remote cleanup and the leftover and archive-coverage checks read records branches and archive tags from the namespace.

Rationale: one code path per operation, so the two layouts cannot drift apart.

### D5 — What of f06acf39 goes

Everything f06acf39 added only to keep records on a code branch is removed:

- the root-marker layout read in `lymcwd` and fabric's per-worktree probe;
- the one-repository clone path and the `lym-bolt` orphan board branch;
- path-limited records commits on the code branch, the temporary-index commit during a merge,
  and the git config key `lym.primeBranch` (removed), a separate key from D1's records-store key `lym.primeRecordsBranch` (new);
- the records strip at every merge, the no-fast-forward rule, the `Fabric-Merge` trailer and the `.lym-anchor` change refusal;
- the merge-stopped-before-records-step lifecycle, its unsettled-merge kind and `landingshed`'s abort-only way forward;
- replay of records-only commits and its `Warp-SHA` rewrite;
- the prime push guard, the `operator_unpushed` and `protected` failed-push classes and their creel notice;
- `statuscommit`'s operation wait and its protected and stalled seams, and the duplicated seams in `battencli` and `loomcli`;
- the `lym-view/` ref teardown and the one-repository exclude seeding;
- the one-repository branches in shared fabric files, the one-repository fixtures in `hubforge`, and their tests;
- the landing test expectation of the `Fabric-Merge` trailer that `a823f323` added.

Rationale: D1 leaves no records on a code branch, so none of this has a case to serve.

### D6 — What of f06acf39 stays

- `ProtectedBranchRefusal` stays in fabric's push classification, and `landingshed` reads it for the parent push.
- A protected-branch refusal stays a named failure that the push retry does not retry, on every branch.
- `gitrepo` keeps the `remote:` lines it appends to a push rejection.

Rationale: these serve the landing push and push diagnostics in both layouts.

### D7 — f06acf39's other two-repository widenings revert

- The merge-state probe reports a merge only; a cherry-pick, revert or rebase makes no status commit skip.
  When `two-repo-merge-probe-regression` lands first, its revert stands and this task removes only what D5 names.
- A deferred failed-push record is not mailed, whether or not it names commits, as before f06acf39.
- Fabric's commit refusal order is the pre-f06acf39 order.

D6's non-retried protected refusal is the one widening kept.
These apply to every hub, since D1 gives a one-repository hub the two-repository local behaviour.

Rationale: each widening served records on a code branch.

### D8 — Fixtures and tests

`hubforge`'s one-repository builder makes one bare origin and seeds it with `lym fabric clone --single-repo <url>`;
it builds a second machine's hub with a plain one-URL clone.
The layout's tests run the two-repository suites' key flows over that builder, plus D3's refusals and D1's namespace placement.

Rationale: a shared builder keeps both layouts on the same flows.

### D9 — Docs and PATTERN

`README.md` and `docs/getting-started.md` describe the one-repository hub as D1 to D4 build it, and its hazards:

- a `git push --mirror` from either store deletes the other's refs on origin;
- a `git clone --mirror` carries the records;
- a bare `git push` in the records store pushes every local tag into the namespace;
- branch protection and rulesets do not cover `refs/lym/`;
- `--single-repo` over the code repository of an existing two-repository hub starts a separate hub.

`docs/overview.md` and `internal/fabricengine/doc.go` drop the records-on-code-branch text.
The PATTERN entries change as `## PATTERN changes` lists;
these backgrounds lose their one-repository text:

- `PATTERN-fabric-boundary`, `PATTERN-lym-files` and `PATTERN-pair-remote`, their one-repository paragraphs;
- `PATTERN-cwd-resolution`, the one-repository `ResolveWithin` line and the worktree-root anchor clause;
- `PATTERN-run-fsm`, the `operator_unpushed` and `protected` paragraph and its `TestPrimePushGuard` pin.

`PATTERN-single-repo-hub`'s background is rewritten to D1 to D4.

Rationale: every f06acf39 doc statement about the old layout becomes false.

### D10 — No conversion of an f06acf39-layout hub

Fabric carries no migration of a hub cloned with f06acf39's layout; D3's refusal of such an origin is the only code that knows it.
The operator cleans the origin as that refusal names, then clones again with D3.

Rationale: the layout landed on 2026-11-19 for the operator's own use, and the operator has ruled out migration code
(`drop-compat-migration-code`, decided 2026-11-19); a refusal that names the cleanup is the cheap kind that entry keeps.

## Constraints

- `PATTERN-fabric-boundary`: every git operation on either store stays in `internal/fabricengine`, with `gitrepo` as its client;
  the layout read and the namespace live in fabric.
- `PATTERN-gitrepo-client-boundary`: the records store's fetch and push run through `gitexec` and `gitrepo` under the remote-refs lock,
  as the weft's do today.
- `PATTERN-cwd-resolution`: `lymcwd` resolves from the `.lym-anchor` at the records root, with no layout case.
- `PATTERN-code-conventions`: the namespace root and its parts are named constants.
- `PATTERN-refusal-way-forward`: each D3 refusal names its way forward and a test reaches it.
- `PATTERN-test-only-code`: D5 and D6 keep no function that only tests reach;
  a general helper f06acf39 added stays only when a production path that remains calls it.
- `PATTERN-pair-remote`, `PATTERN-run-fsm` and `PATTERN-owner-machine` hold unchanged for a one-repository hub,
  since its pairs run the two-repository paths.

## Auto-mode assumptions

- The namespace is `refs/lym/heads/` and `refs/lym/tags/`, not ordinary branches under a prefix.
- The layout is the records store's refspec, not a separate config key.
- No hub depends on f06acf39's layout, so D10 needs no conversion.

## Open risks

- Peer run `two-repo-merge-probe-regression` is changing `internal/fabricengine/onerepository.go`,
  `mergestateactive.go`, `recordsduringmerge.go`, `clone_onerepository.go` and `anchormarker.go`, which D5 deletes or rewrites;
  the probe cache it builds is removed here.
  The parent may order the two runs or narrow the peer to its merge-probe revert.
- The parent project `single-repo-hub` is titled for records on each code branch and the board on `lym-bolt`, which this design replaces;
  the parent may retitle it.

## Acceptance criteria

- D1, D3: `lym fabric clone --single-repo` over a bare origin with no namespace puts the board and the prime's records branch
  under `refs/lym/heads/`, leaves every `refs/heads/` ref at its prior tip, and builds a hub whose layout matches a two-repository hub's:
  the records sibling, `_lym` junctions and `_bolt`.
- D1: a default `git clone` of that origin fetches no ref under `refs/lym/`, and the hub's records store holds no code branch.
- D3: a one-URL clone of an origin holding the namespace builds the same layout on a second machine, with the board's content.
- D1, D3: a `--single-repo` clone of an origin whose code default branch is `master` puts the board at `refs/lym/heads/main`
  and the prime's records branch at `refs/lym/heads/master-weft`;
  a one-URL clone of that origin on a second machine checks out that board with its content;
  and the orphan sweep in that hub keeps `master-weft`, also after `lym fabric checkout` moves the prime off `master`.
- D3: a one-URL clone refuses an origin holding the namespaced board branch with `--single-repo`,
  an origin whose default branch holds `.lym-warp` with `--single-repo`, a second URL with `--single-repo`,
  an origin holding `lym-bolt`, and with `--single-repo` an origin whose default branch tracks a root `.lym-anchor` or `_lym`,
  each before creating anything and each naming its way forward.
- D3: a one-URL clone of a two-repository weft whose board tracks the legacy root `.lym-anchor` clones as today.
- D3: a seed push that fails leaves no hub directory, names its cause, and leaves no namespaced board branch on origin;
  a rerun of `--single-repo`, with the same or another `--subpath`, replaces the leftover records branch and succeeds.
- D3: a `--single-repo` clone whose board push loses its lease returns the records branch to the tip it observed,
  and names the one-URL clone of the other hub as its way forward.
- D2, D4: in a one-repository hub, a pair add, push and teardown place its records branch at `refs/lym/heads/<branch>-weft`
  and its archive tag under `refs/lym/tags/archive/`, add no ref under `refs/heads/` or `refs/tags/` beyond the code branch,
  and teardown removes the namespaced records branch.
- D4: in either layout, a teardown of a never-pushed records branch records no remote deletion.
- D2, D4: in a one-repository hub, origin reconcile and remote cleanup in the records store see the namespaced records branches
  and archive tags, and no code branch.
- D2: a records branch's upstream in a one-repository hub names its namespaced ref, and the branch reads as in sync after a push.
- D2: the layout read of a path with no records store logs nothing and reads two repositories;
  a second read of an existing store in the same process runs no git;
  a store created after a missing-store read reads one repository.
- D2, D5: a code branch merged by fabric in a one-repository hub has no `_lym` path in its tree,
  and its merge commit carries no `Fabric-Merge` trailer.
- D5: a records commit in the prime commits on any branch the prime is on.
- D5, D6: a push refused as a protected branch records the ordinary rejection class, and the retry does not retry it.
- D7: a status commit during a cherry-pick in the records worktree commits instead of skipping.
- D7: a deferred failed-push record that names commits sends no mail.
- D8, D10: `hubforge`'s one-repository builder produces a hub over one bare origin that the D1 and D4 criteria run against,
  and D3's f06acf39-origin refusal is reached over a bare origin holding `lym-bolt`.
- D9: Review: `README.md` and `docs/getting-started.md` describe the namespace layout and D9's hazards,
  and no doc or PATTERN text describes records on a code branch or `lym-bolt`.

## PATTERN changes

- `PATTERN-single-repo-hub` — changes: Running a hub as one repository: a code store and a records store of one origin, records and the board under origin's `refs/lym/`, mapped by the records store's refspecs; only fabric reads the layout.
- `PATTERN-run-fsm` — changes: Reading or acting on a run's lifecycle state: only `internal/runfsm`'s states and act rule; a transition commits and pushes before the run acts; a run whose record holds another machine's takeover hands over, never replays.
- `PATTERN-cwd-resolution` — changes: Resolving a cwd, worktree root or anchor: only `lymcwd`, from the worktree's weft-root `.lym-anchor`; only `fabricengine` writes or git-reads it; engines get absolute paths, never import lymcwd; hub containers and workspace files stay hub-reachable only.

## Notes for the plan writer

- f06acf39's own commit message lists its parts and files by concern; reverting its one-repository parts against D6's keep list
  may be cheaper than editing forward.
- The clone CLI's argument order is `<weft-url> [<warp-url>]`; D3's one-URL probe runs before the existing weft-binding probe.
- The lens review traced the records store's remote paths D2 must cover to these files;
  the list is a head start, not a completeness claim:
  - `internal/gitrepo/`: `push.go`, `pushretry.go`, `pushdestination.go`;
  - `internal/fabricengine/`: `originreconcile.go`, `cleanupremotewarp.go`, `remoteleftover.go`, `archive.go`, `recordsprobe.go`.
