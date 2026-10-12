# Card 3 — loom-peers

**Create:**
- `internal/loomcli/peers_test.go`
- `internal/loomcli#peerLine`
- `internal/loomcli#peersDeps`
- `internal/loomcli#peersVerb`
- `internal/loomcli#peersShowVerb`
- `internal/loomcli#hubPeersDeps`
- `internal/loomcli#loomCLI.peersCmd`
- `internal/loomcli#TestPeersVerb`
- `internal/loomcli#TestPeersShowVerb`
- `internal/loomcli/peers_integration_test.go`
- `internal/loomcli#TestHubPeersDeps_OverRealHub`

**Edit:**
- `internal/loomcli#inReviewGroup`
- `internal/loomcli#Command`
- `internal/loomcli/doc.go`
- `docs/overview.md`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/fabricengine#Fabric.CommitLog`

**Intent:**
Add D6's read-only verb, so an agent in one pair can read the other in-flight runs' records and the parent's landed commits without forming another worktree's path itself.

The verb lives in a new file `peers.go`.
`peersVerb` and `peersShowVerb` are pure bodies over `peersDeps`, whose fields are closures:
- `board`: the board's tasks, with `false` when the hub has no board dir.
- `caller`: the caller's own slug, empty from the prime.
- `readPeer`: fills one `peerLine` for a task.
- `parentCommits`: the caller's parent commits; nil from the prime.
- `readRecord`: returns one peer record's content.

`hubPeersDeps` fills them for a resolved location:
- the board through `boardengine.OpenHub(location.HubPath).ListTasksFull`, after a stat of `fabricengine.BoardDir(location.HubPath)`.
- the prime test through `fabricengine.PrimeName`.
- a peer's pair path through `fabricengine.WorktreePath(location, slug)` and `lyxcwd.ResolveWorktree`, and its parent branch through `fabricengine.ReadOriginFor(location, slug)`.
- its changed paths through the peer's `fabricengine.Open` handle: `MergeBase(parent)`, `HeadSHA` and `ChangedPaths(base, head)`.
- its records by stat of `loomengine.DiscussionDecisionRecord(peer)` and of `00-overview.md` under `planparser.PlanDir(peer.AnchorPath())`.
- the caller's parent commits through `readRecordedParentBranch`, `MergeBase` and card 2's `CommitLog(base, parent)`.

Every git read goes through `internal/fabricengine`.

The peer set is every board task whose `Status` `boardengine.IsRunStatus` accepts, whatever the state, so `awaiting` and `blocked` runs count, with the caller's slug excluded.
`peersVerb` prints one success envelope.
Its `peers` key holds one object per peer: `slug`, `title`, `status`, `parent_branch`, `changed_paths`, `decision_record` and `plan` (booleans), and `unreadable`, the list of what could not be read.
Its `parent_commits` key holds `<short sha> <subject>` lines and is present only for a task pair.
A peer whose pair, origin record, branch or records are missing or unreadable is listed with that reason in `unreadable`, never dropped and never an error.
With no board dir both verbs refuse: "loom: peers: this hub has no board, so no run is in flight beside this one; way forward: nothing to compare, and a standalone run's review stays with its parent".

`lyx loom peers show <slug> --record decision|plan` prints the record's content under a `content` key, with the plan given as its `00-overview.md`.
Each refusal names its way forward:
- an unknown slug: "`lyx loom peers` lists the in-flight peers".
- a slug whose status is no run status: the same.
- the caller's own slug: "read your own records in this worktree".
- an unknown record name: "pass --record decision or --record plan".
- a record the peer does not have yet: "`lyx loom peers` shows which records exist".

Bound: the verb reads only tasks whose board status is a run status, and for each only its title, status and parent branch, its decision record, its plan and its changed-path list, plus the caller's parent commit list; it writes nothing.

`peersCmd` builds a `peers` group, which runs the list as its own `RunE`, with `show` beneath it.
Both carry `clihelp.AudienceRole`, and the group has its own `PersistentPreRunE` that resolves the location through `lyxcwd.CwdFrom` and `lyxcwd.Resolve`, the way the `decision` group's pre-run does.
`inReviewGroup` also matches the `peers` group, so the loom parent's pre-run never arms it, and its doc comment says so.
`Command` adds `c.peersCmd()`.
Geometry resolves through `lyxcwd`, not `internal/cliwire`: see the Shared Decision `peers-geometry`.

Tests, both Tier 1 over fake `peersDeps`:
- `TestPeersVerb`: running, awaiting and blocked peers are listed, with a task holding no run status (nil, `done`, a hand-set word) and the caller's own slug left out. From the prime, every run-status task is listed and `parent_commits` is absent. A peer whose read reports a missing pair, a missing branch and a missing record carries all three in `unreadable`. The caller's parent commits appear in order. No board dir refuses with the no-board message.
- `TestPeersShowVerb`: a decision record and a plan are printed. Each refusal above reaches its row: an unknown slug, a non-run-status slug, the caller's own slug, an unknown record name, an absent record and no board dir. Each asserts its way-forward text.

`TestHubPeersDeps_OverRealHub` (`integration` tag) drives the real `hubPeersDeps` closures over a `hubforge.CopyHub` hub with pairs added through `hubforge.AddPair`, as `TestLandingDeps_ConfigNoticeSeamsOverRealHub` does (`PATTERN-hubforge-fixtures`).
- The caller pair and a peer pair each commit after forking. The parent then gains two commits.
- The peer's line lists exactly its committed paths against its merge base with its parent, and `decision_record` true once a decision record is written in it.
- The caller's `parent_commits` are the parent's two new commits, newest first, which pins card 2's range direction: base to parent tip.
- A board task whose pair directory was never created lists `unreadable` naming the pair.
- A peer whose origin record is removed lists `unreadable` naming the branch, and its line is kept.

`refusal-spec.md`'s loom section gains a row for each refusal above.
`docs/overview.md`'s loom verb list adds `peers|peers show`, and `doc.go` gains a paragraph on the verb, its peer set and its bound.
`test-budget.yaml` raises `internal/loomcli` by three.

**ImpactSummary:** A new read-only verb group under `lyx loom`; `inReviewGroup` widens to skip it, which changes no existing verb's pre-run.
