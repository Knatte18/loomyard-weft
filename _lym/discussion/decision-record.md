## Goal

Remove f06acf39's records-on-code-branch layout and revert its two-repository widenings

Takes: f06acf39's own commit message and file list as the removal map, reverted against D6's keep list; c83fec49's cached layout probe and its tests, which go with onerepository.go; fabricengine.FailedPushWayForward and its callers in pairteardown, loomcli and landingshed, weighed against D7's failed-push mail revert before any of it is deleted
Delivers: a fabric with only the two-repository layout: no root-marker layout read in lymcwd or fabric, no one-repository clone path, no lym-bolt, no --single-repo flag, no hubforge NewOneRepoHub, no path-limited or mid-merge records commits, no records strip, Fabric-Merge trailer or no-fast-forward rule, no records-only replay, no prime push guard or operator_unpushed and protected classes, no statuscommit operation wait or its battencli and loomcli seams, no lym-view ref teardown, no lym.primeBranch key; the merge-state probe reports a merge only, a deferred failed-push record is not mailed, and fabric's commit refusal order is the pre-f06acf39 one; docs/overview.md and internal/fabricengine/doc.go hold no records-on-code-branch text, and the PATTERN-run-fsm and PATTERN-cwd-resolution entries and backgrounds lose their one-repository text
Exports: nothing new; it removes lymcwd.OneRepository, fabricengine.MergeStateOperation, IsPrimeBranchRefusal, IsPushProtected, MidMergeRecordsUnsettled, PushOneRepositoryClone, CloneOptions.SingleRepo, BoltBranchName, gitrepo.MergeStartNoFastForward and hubforge.NewOneRepoHub, and keeps fabricengine.ProtectedBranchRefusal and gitrepo's remote: lines on a push rejection

## Scope

The part changes these packages:

- internal/fabricengine (about 6900 changed lines)
- internal/fabriccli (about 375 changed lines)
- internal/statuscommit (about 230 changed lines)
- internal/lymcwd (about 145 changed lines)
- internal/loomcli (about 140 changed lines)
- internal/hubforge (about 140 changed lines)
- internal/battencli (about 100 changed lines)
- internal/landingshed (about 75 changed lines)
- internal/boardengine (about 67 changed lines)
- internal/runfsm (about 35 changed lines)
- internal/gitrepo (about 30 changed lines)
- cmd/lym (about 8 changed lines)
- internal/pairteardown (about 6 changed lines)
- docs (about 5 changed lines)
- pattern (about 40 changed lines)

## Decisions

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

### Added after Discussion (parent, 2026-11-19): Peer merge-probe run has landed

Decision: two-repo-merge-probe-regression landed on main as c83fec49 before this split; the removal part starts from that main and deletes or rewrites its probe cache as D5 says. No ordering is needed.

Rationale: The peer is no longer running, so there is nothing to order or narrow.

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

## Acceptance criteria

- D5: a records commit in the prime commits on any branch the prime is on.
- D5, D6: a push refused as a protected branch records the ordinary rejection class, and the retry does not retry it.
- D7: a status commit during a cherry-pick in the records worktree commits instead of skipping.
- D7: a deferred failed-push record that names commits sends no mail.

## PATTERN changes

- `PATTERN-run-fsm` — changes: Reading or acting on a run's lifecycle state: only `internal/runfsm`'s states and act rule; a transition commits and pushes before the run acts; a run whose record holds another machine's takeover hands over, never replays.
- `PATTERN-cwd-resolution` — changes: Resolving a cwd, worktree root or anchor: only `lymcwd`, from the worktree's weft-root `.lym-anchor`; only `fabricengine` writes or git-reads it; engines get absolute paths, never import lymcwd; hub containers and workspace files stay hub-reachable only.
