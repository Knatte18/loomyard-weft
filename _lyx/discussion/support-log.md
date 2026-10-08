# Support log: git-robustness

## Interview

Autonomous mode: no operator answered; every pick below is an auto-pick, recorded in the question ledger.

1. Read the board entry `git-robustness` and the entries it touches: `publish-recovery` (staged merge-in recovery, dirty-tree refusal), `lyx-cleanup` (batten's cleanup verb for finished statuses), `verify-watchdog` (Windows process-tree kill), `fabric-pushes` (weft push retry).
   Those set the scope edges.
2. Explored `internal/gitrepo`: `lookupObjectRetrying` wraps single lookups only; `Worktree.Status()` in `worktree.go`, `Log` walks in `HeadContains` and `PathRevisions`, and tree diffs run their object reads inside go-git, outside any wrapper.
   Settled on wrapping the whole read (D1).
3. Explored git execution: every production git spawn goes through `gitexec.runCore`; `IsTransportFailure` already classifies remote subcommands by stderr.
   Settled on the timeout in `runCore` and a timed-out command classified as a transport failure (D3).
4. Checked the hub store: `git count-objects -v` shows about 8,600 loose objects and 36 packs; `gitkit` and `hubforge` fixtures already set `gc.auto=0` and `maintenance.auto=false`.
   Settled on `gc.auto=0` in the stores plus a foreground `gc --auto` at pair removal (D2).
5. Explored batten: `battenPreRun` already refuses a done slug through `doneSlugRefusal`; `--window` returns from `runInWindow` before any status read, which is how #447 opened an empty window.
   Settled on running the same check before the window opens (D4).
6. Explored `HeadContains`'s one caller, `fabricengine.StencilSource`: a wrong answer changes only a drift log line.
   Settled on a committer-time-bounded go-git walk (D5).

## Rejected alternatives

- D1, wrapping only `Status()` calls: the stale index also breaks `Log` walks and tree diffs, and the board asks for one place.
- D1, re-opening the go-git handle on failure: `internal/gitrepo/doc.go` records that `Reindex()` fully repairs the handle, so a re-open buys nothing.
- D1, proactive reindex before every read: the fingerprint check costs a directory read per call, and the doc's measured reasoning favors the reactive gate.
- D2, `-c gc.auto=0` on lyx's own git calls only: agents' and the operator's commits in the pairs would still start detached repacks.
- D2, pruning unreferenced loose objects with a short expiry at landing: a short prune expiry races in-flight writes in other pairs; git's default expiry stays.
- D2, full `git gc` at every pair removal: repacks everything each time; `gc --auto` does work only when needed.
- D2, running housekeeping inside `destroy.go`: it removes nothing the destruction gate's checks protect, and the gate's ownership model does not fit an object-store prune.
- D3, `http.lowSpeedLimit` alone: does not cover SSH remotes or credential-helper hangs.
- D3, per-caller contexts: every caller would need wiring and a new one could forget it; `runCore` covers all.
- D3, operator-configurable timeouts: no requirement asks for it.
- D4, treating a done slug as free (archive the status, start fresh): the board Goal says refuse, the finished-status cleanup is `lyx-cleanup`'s, and a fresh start would also need the old task branch removed.
- D5, `git merge-base --is-ancestor` through the CLI: fast with commit-graph, but moves a local read to the CLI against `PATTERN-gitrepo-client-boundary` and costs a spawn.
- D5, a depth bound: a fixed depth gives wrong answers on a busy branch and still walks far on a sparse one; committer time tracks the build commit directly.
- D5, go-git's `MergeBase` or `Commit.IsAncestor`: both walk without a bound.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): Which go-git reads does the retry cover? All of them, ref reads included, through one helper that is `goGit()`'s only caller. Reason: one enforceable place, and the board asks for every object-resolving call.
- Q2 (resolved, auto-pick): How does lyx keep repacks off live steps? `gc.auto=0` in both hub stores, foreground `gc --auto` after each pair removal. Reason: store config reaches every git caller; removal is a natural quiet point; D1 covers the remaining overlap with other pairs.
- Q3 (resolved, auto-pick): Where is `gc.auto=0` applied on an existing hub? At the next pair create and before every housekeeping run. Reason: no separate migration step; both points run soon on any active hub.
- Q4 (resolved, auto-pick): Network timeout values: 1000 B/s for 60 s and a 10-minute wall clock. Reason: a dead connection fails in about a minute; the wall clock covers transports the low-speed limit misses, with room for a large fetch.
- Q5 (resolved, auto-pick): Is a timeout a transport failure? Yes, so existing best-effort and transient paths take over. `pull` joins the classified subcommands.
- Q6 (resolved, auto-pick): Does a done slug refuse or run fresh? Refuse, on both the windowed and windowless paths; the refusal also names choosing another slug. Reason: board Goal; `lyx-cleanup` owns status removal. The #447 note's operator expectation (reuse) is recorded as an open tension in the decision record's assumptions.
- Q7 (resolved, auto-pick): How is `HeadContains` bounded? Committer-time window, 24 hours of slack. Reason: cost scales with commits since the build; a skew-induced wrong answer changes only a log line.
- Q8 (resolved, auto-pick): New PATTERN entries? Two: the go-git retry chokepoint and hub-store gc ownership. The network timeout needs none, since `runCore` is already every call's path.
