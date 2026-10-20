# Card 15 — origin-reconcile-judge

**Create:**
- `plan:internal/fabricengine#ReconcileScope` -> `type ReconcileScope struct`
- `plan:internal/fabricengine#ReconcileOptions` -> `type ReconcileOptions struct`
- `plan:internal/fabricengine#EndedTeardown` -> `type EndedTeardown struct`
- `plan:internal/fabricengine#TeardownReport` -> `type TeardownReport struct`
- `plan:internal/fabricengine#OriginSeams` -> `type OriginSeams struct`
- `plan:internal/fabricengine#BranchState` -> `type BranchState string`
- `plan:internal/fabricengine#BranchLevel` -> `const BranchLevel BranchState = "level"`
- `plan:internal/fabricengine#BranchFastForward` -> `const BranchFastForward BranchState = "fast_forward"`
- `plan:internal/fabricengine#BranchAhead` -> `const BranchAhead BranchState = "ahead"`
- `plan:internal/fabricengine#BranchRewound` -> `const BranchRewound BranchState = "rewound"`
- `plan:internal/fabricengine#BranchRewritten` -> `const BranchRewritten BranchState = "rewritten"`
- `plan:internal/fabricengine#BranchDiverged` -> `const BranchDiverged BranchState = "diverged"`
- `plan:internal/fabricengine#BranchNeedsReplay` -> `const BranchNeedsReplay BranchState = "needs_replay"`
- `plan:internal/fabricengine#BranchGone` -> `const BranchGone BranchState = "gone"`
- `plan:internal/fabricengine#BranchDirty` -> `const BranchDirty BranchState = "dirty"`
- `plan:internal/fabricengine#BranchBusy` -> `const BranchBusy BranchState = "busy"`
- `plan:internal/fabricengine#BranchLive` -> `const BranchLive BranchState = "live"`
- `plan:internal/fabricengine#BranchOutcome` -> `type BranchOutcome struct`
- `plan:internal/fabricengine#EntryState` -> `type EntryState string`
- `plan:internal/fabricengine#EntryActive` -> `const EntryActive EntryState = "active"`
- `plan:internal/fabricengine#EntryEnded` -> `const EntryEnded EntryState = "ended"`
- `plan:internal/fabricengine#EntryStopped` -> `const EntryStopped EntryState = "stopped"`
- `plan:internal/fabricengine#EntryOutcome` -> `type EntryOutcome struct`
- `plan:internal/fabricengine#OriginReport` -> `type OriginReport struct`
- `plan:internal/fabricengine#OriginReport.Summary` -> `func (r OriginReport) Summary() string`
- `plan:internal/fabricengine#entryFacts` -> `type entryFacts struct`
- `plan:internal/fabricengine#classifyEntry` -> `func classifyEntry(f entryFacts) EntryState`
- `plan:internal/fabricengine#originCallHook` -> `var originCallHook func(args []string)`
- `plan:internal/fabricengine#lsRemotePair` -> `func lsRemotePair(warpDir, recordsDir string, opts ReconcileOptions) (warp, records map[string]string, err error)`
- `plan:internal/fabricengine#fetchMovedHeads` -> `func fetchMovedHeads(dir string, heads map[string]string, judgeOnly bool) error`
- `plan:internal/fabricengine#judgeBranch` -> `func judgeBranch(repoDir, branch, originHead string, records bool) (BranchOutcome, error)`
- `plan:internal/fabricengine#ReconcileOrigin` -> `func ReconcileOrigin(l *lyxcwd.Location, scope ReconcileScope, seams OriginSeams, opts ReconcileOptions) (res OriginReport, err error)`
- `plan:internal/fabricengine#SetOriginCallHookForTest` -> `func SetOriginCallHookForTest(hook func(args []string)) (restore func())`
- `plan:internal/fabricengine_test#TestReconcileOrigin_Judge` -> `func TestReconcileOrigin_Judge(t *testing.T)`

**Edit:**
- `internal/fabricengine/doc.go`
- `PATTERN.md`

**Uses:**
- `plan:internal/fabricengine#trackingReflog`
- `plan:internal/fabricengine#classifyRewrite`
- `internal/fabricengine#ReadOriginFor`

**Intent:** D1, D2, D4's judgment and D7: the origin reconcile entry point in fabricengine (new file `originreconcile.go`), judging only; the moves and the ended entries follow in cards 16 and 17.

`ReconcileOrigin` takes the hub location, a `ReconcileScope` and the injected `OriginSeams`, and returns an `OriginReport` embedding `MutationRecord` (PATTERN-mutation-record) with one `BranchOutcome` per judged branch and one `EntryOutcome` per judged entry.
`ReconcileScope.Run` empty is the hub scope (Bolt, the prime's warp and weft branches, every local pair's warp and weft branches); a slug is the run scope (Bolt, the run's parent branch on both sides read from the pair's origin record, and the run's own pair).
`ReconcileScope.JudgeOnly` is `lyx cleanup`'s dry run.
`OriginSeams` carries what fabricengine reaches only by injection (D1): `LivePair func(slug string) (bool, error)` (driver strand, run and loop locks, the hub pause), `PrimeParentLockFree func() (bool, error)` (one read of the prime branch's parent lock record) and `Teardown func(EndedTeardown) (TeardownReport, error)` (pairteardown's local mode); `EndedTeardown` names the slug and whether the force case applies, and `TeardownReport` carries the removal's tags, kept reasons, busy flag and error.
`ReconcileOptions` holds the retry bound, the retry interval, the per-attempt deadline and the sleep, all injectable; the zero value makes one attempt under a 20-second deadline, and the wake caller (card 23) passes 2 minutes, 10 seconds and 20 seconds.

The network work is exactly D1's: `lsRemotePair` runs `git ls-remote origin` for the warp repo and the records repo in parallel, each under the attempt deadline, retrying the pair together while either fails with a transport failure and the bound allows; then `fetchMovedHeads` fetches, in one fetch per repo, only the heads whose origin SHA differs from the local tracking ref, into the tracking refs, or in judge-only mode into no ref (`git fetch --refmap= --no-write-fetch-head origin <sha>...`), so the dry run moves no ref.
`originCallHook`, nil in production, sees each remote call's arguments, for the test that counts them; `SetOriginCallHookForTest` in `export_test.go` swaps it, package-global, so the tests that set it run without `t.Parallel`.
A transport failure through the bound is the report's `Unreachable` with nothing judged against origin and nothing moved; every caller carries on with a warning (Auto-mode assumptions).
An ls-remote failing with an error that is not a transport failure (an auth refusal, a missing remote) is retried by no bound and returned as `ReconcileOrigin`'s error, naming the repo and git's error line, with nothing judged or moved; so is a fetch that fails after both ls-remote calls answered, a transport failure of it included, since the heads it was to fetch are known to exist; each caller's card says what its error does.

`judgeBranch` judges one local branch with an origin tracking ref against origin's head: **level**, **fast_forward** (local tip an ancestor of origin's head), **ahead**, **rewound** and **rewritten** (by `classifyRewrite` over `trackingReflog`, rewound taking precedence over ahead), **diverged**, or **gone** (no origin head for its upstream).
`BranchOutcome` carries the branch, its repo side, its state, the conflicting or dirty paths, the dropped commits of a rewind, and for **busy** which lock was held, with `PushLockHeld` set when it was the push lock, so a call site tells a push parked under a pause (D8's `lyx resume <slug>` way forward) from a held write lock.
A records branch (a weft or Bolt) that neither side contains reads **needs_replay** here, which card 16's moving reconcile replaces with its replay's result, and which the judge-only report keeps.
`classifyEntry` is D7's one switch over an entry's facts: `ended` (from card 17's ended set) first, carrying a busy flag when its driver or branch is busy; then `stopped` for a diverged, rewound, rewritten, dirty, gone or busy branch of it, a live pair whose branch origin moved past or diverged from, or a D3 stop; otherwise `active`; `machine-handover`'s owner case slots ahead of `active`.
A pair with no batten run record is never classified, only its branches judged.
`OriginReport.Summary` renders the report in a few lines (moved, stopped, ended, unreachable) for the orch's start note and the wake notice; each stop names the branch and the way forward its state takes (D4's: `ResetPairCode`'s way forward for rewound, `lyx fabric pull` in the pair's worktree for rewritten, the conflicting paths for diverged, `lyx resume <slug>` for busy).
`doc.go` gains an Origin reconcile section, and `PATTERN.md` gains the `PATTERN-origin-reconcile` entry line in its "Fabric and git" section, worded exactly as the overview's PATTERN changes bullet, within the entry line cap.

`TestReconcileOrigin_Judge` (integration, a `hubforge` hub with a pair and second clones of both bares as the other machine; seams faked; run without `t.Parallel` since it sets the call hook) rows: origin moved on a weft, Bolt and warp `main` with nothing local reads fast_forward for each (judging only); a code branch with local and origin commits reads diverged; a task branch rewound on origin by a leased update reads rewound, with and without an unpushed local commit, before and after its tracking ref was fetched to origin's head, the fetch made once by `Fabric.Pull` and once by a judge-only reconcile, after which the next reconcile still reads it rewound; a branch whose origin head is an ancestor of an earlier tracking-ref value while its tip holds none of that value's lacking commits and neither tip contains the other reads diverged; a warp upstream rewritten to a head no earlier value contains, with no unpushed commit, reads rewritten and names `lyx fabric pull`; a deleted upstream reads gone; judge-only over a moved weft fetches objects and leaves every ref, tracking refs included, unchanged, and reports a diverged Bolt as needs_replay; an origin URL of `http://127.0.0.1:1/x` on the records repo, which git refuses with `connection refused` (a phrase `gitexec.IsTransportFailure` matches), reports `Unreachable`; with injected options and a sleep that restores the records repo's real origin URL after the first attempt, the ls-remote pair whose first attempt was refused and whose second answers judges normally, and with a sleep that restores nothing the pair is `Unreachable` after the bound's attempts; an origin URL naming a path that holds no repository (not a transport failure) returns an error with no retry; a fetch that fails after both ls-remote calls answered (the records bare's moved head deleted from the bare between the ls-remote and the fetch through the call hook) returns an error with nothing moved; the call hook counts exactly two ls-remote calls and fetches naming only moved heads; `classifyEntry`'s precedence (ended over every branch cause, a D3 stop over active) as table rows.

**ImpactSummary:** Adds one new entry point and its types in a new fabricengine file, plus a doc section and one PATTERN line; no existing caller or behavior changes.

**Commit:** 15: fabricengine: the origin reconcile judges branches against origin
