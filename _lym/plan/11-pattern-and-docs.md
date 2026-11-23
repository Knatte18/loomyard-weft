# Card 11 — pattern-and-docs

**Prosa:**
- `PATTERN.md`
- `pattern/PATTERN-run-fsm.md`
- `pattern/PATTERN-pair-remote.md`
- `pattern/PATTERN-owner-machine.md`
- `pattern/PATTERN-daemon-wakeups.md`
- `internal/heddleengine/glossary.md`
- `docs/overview.md`

**Intent:**
State the Decisions in the three PATTERN entries, their backgrounds, the glossary and the overview doc.

`PATTERN.md` replaces each entry's line with its declared text, keeping its tier mark and background link:

```markdown
- `PATTERN-run-fsm` — Reading or acting on a run's lifecycle state: only `internal/runfsm`'s states and act rule; a transition commits and the run acts at once, a background pusher bringing every commit to origin in order; a run whose record holds another machine's takeover hands over, never replays. — [background](pattern/PATTERN-run-fsm.md)
```

```markdown
- `PATTERN-pair-remote` — Pushing run records or tearing down a pair: the code branch goes along after the records side, from the worktree's background pusher, which no transition waits for; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`. (test) — [background](pattern/PATTERN-pair-remote.md)
```

```markdown
- `PATTERN-owner-machine` — Starting, resuming, pushing or landing a run, a Finalize past its merge included, or picking a pair's parent: only on the entry's owner machine, read via hubgeom's `ParentOf` and Bolt's machine registry; a loom pusher reconciles first; after absence, a catch-up from origin first. — [background](pattern/PATTERN-owner-machine.md)
```

`pattern/PATTERN-run-fsm.md`:
- Its opening line takes the new rule.
- "A transition pushes before the run acts" becomes "A transition commits and the run acts":
  a transition commits its records with its `Run-Transition` and `Warp-SHA` trailers and starts the worktree's pusher;
  it pushes, fetches and reconciles nothing and takes no gate admission;
  the pusher, one per records worktree (`internal/runpusher`), pushes records side first and code after, in order, none left out;
  its restarts are the next commit, the dobby's tick and a resume. Held by `TestNew_OrdinaryPath`, `TestNewCommitStatusSeam` and `TestRun`.
- "An owed push holds the run" becomes "An owed push no longer holds the run":
  the pusher retries it in its backoff and writes no owed record, so `PushOwed` stays unset while it retries;
  a refused push's record keeps its refusal and way forward. Held by `TestPushPairAnchored_OwedUnrecorded` and `TestRun`.
- "The start rows commit only" goes.
- "A takeover hands over" adds that a takeover re-runs from what origin holds, that the pair's pusher hands its run over,
  and that the prime's pusher hands over each batten run its unpushed commits name, archiving them and replaying the rest.
  Held by `TestRejectedRecordsPush_ReadsTheTakeover`, `TestPushRun_HandsOverOnATakeover` and `TestAdoptIntegration_SecondCloneTakesOver`.
- "A missing code tip waits" adds that a run whose pair stands here and holds the tip is not held, since its pusher owes it.

`pattern/PATTERN-pair-remote.md`:
- Its opening line takes the new rule.
- "Push both sides" says the pusher pushes the records side and then the code side;
  no transition waits; an owed push is retried in the pusher, and a refused one leaves its record and notice and stops the pusher
  until the record's way forward clears it; the start-row line goes.
- "Pair teardown" adds that Worktree-Teardown pushes the pair to completion with the prime lock released
  and removes it only while origin's tracking refs hold both tips.

`pattern/PATTERN-owner-machine.md`:
- Its opening line takes the new rule.
- The owner-machine bullet on a loom run's commit and push says a pair's pusher runs the run-scope reconcile before each push,
  a pause covering the run stopping its push; the commit runs no check.
- "Catch-up after an absence" and "Enforcement" say the pusher's reconcile, not the commit's.

`pattern/PATTERN-daemon-wakeups.md`'s "What counts" list replaces the owed transition push's retry in `internal/statuscommit/statuscommit.go`
with the pusher's owed retry in `internal/runpusher`: it waits from one second, doubling to a minute while the push stays owed.

`internal/heddleengine/glossary.md` gains, in its list:
`- **pusher**: the background process that brings one worktree's run commits to origin in order.`

`docs/overview.md`:
- the `commit-records` line says the verb commits and starts the pair's pusher;
- the `internal/statuscommit` line says it skips while mid-merge, makes a commit's failure a hard error and starts the worktree's pusher;
- the package listing gains `internal/runpusher/`, the pusher of a records worktree.
