# Fixer report — discussion round 2

Every finding's premise held against the code (`shuttleengines.go:222-228`, `configengine/doc.go:59`, `run.go:581`/`642`/`969`, `scrub.go:9`, `reedengine/lifecycle.go:456`, `shedadapters/sessionusage.go:107`, `reportengine/cost.go:2`, `orchengine/state.go:88`, `claudeengine/settings.go`, `selfexec/binary.go:19`).

## Changes

- F1 (fixed as proposed, then widened in the self-check): D6 gains a paragraph on how the known directories reach readers.
  Every claude engine that reads or searches them is built with the hub's `Dirs` (`Board` and `Anchor`), `RecordReaders` takes those `Dirs`, and the hub's worktree anchors are a told list the caller resolves.
  Outside a hub a reader reads the process's `CLAUDE_CONFIG_DIR` and the default; the old tokencount-outside-a-hub sentence folded into it.
  New D6 criterion: a record reader built for a hub finds a session on a listed account with its worktree and record gone.
- F2 (fixed as proposed): D6's third bullet names `LocateSessions`, serving the session-end usage recorder and `tools/tokencount`, and states that `lym report` reads the usage records.
  Scope bullet 5 and the two D6 `lym report` criteria are rewritten; the tokencount criterion keeps the `env` directory and once-only count.
- F3 (fixed as proposed): a new D2 bullet has the resume check resolve the directory by D2's rules before it stats, and `Prepare` records what the same search finds.
  D6's first bullet points the resume check at D2's resume rules. New D2 criterion added.
- F4 (fixed as proposed): D2 names the shuttle run records, reed strands and the orch's state record, passed to `Prepare` and `OpenLaunch` as told data; D3 step 1 refers to the told ids. New D2 criterion added.
- F5 (fixed as proposed, plus consequences): a new D4 bullet adds `CLAUDE_CONFIG_DIR` to `ScrubbedEnv`.
  The no-record fallback in D2 and D6 drops the process's value, since panes no longer inherit it. New D4 criterion added.
- F6 and F13 (fixed differently): the proposed rule, which raises any account whose count over weight is below the others' lowest, breaks the counter in normal operation.
  The next account due always sits below the others' lowest.
  With weights 2 and 1: after A's first pick, B (0) is raised to 0.5, ties A, and loses to the earlier entry; this repeats, so A takes every pick.
  Instead the counter file stores each name's weight and the names listed at the last pick.
  Only an account newly listed, re-added, or with a raised weight is raised, to the lowest count over weight among the other accounts listed at the last pick.
  With no such account, every count starts at 0, which covers F13.
  The D3 seeding criterion is rewritten and F6's criterion added.
- F7 (fixed as proposed, plus one addition): the verb takes the cleaned directory beside the name, the usage file stores it, and a file naming another directory is replaced whole.
  Added: the verb also refuses a directory that differs from the named entry's.
  Without that, a session launched before the name was reused would replace the new directory's file with the old directory's figures.
  D5's rationale is rewritten; new D5 criterion added and the refusal criterion extended.
- F8 (fixed as proposed): D1 reads each tracked layer's own document; the D1 criterion adds "also when the machine file lists accounts".
- F9 (fixed, way forward corrected in the self-check): D2 keeps a logged-out record.
  The way forward is to log the directory in again or write the record with another `config_dir`, since removing the record re-finds the same directory.
- F10 (fixed as proposed): both D2 resume criteria state that `claude.accounts` lists an account.
- F11 (fixed as proposed): D6 states that a reader whose list fails to load logs the refusal and reads the remaining known directories.
- F12 (fixed differently in the self-check): D5 names the binary through `selfexec.LymBinary`, and the rendered command changes to the worktree's absolute path known at launch.
  The proposed `cd "${CLAUDE_PROJECT_DIR:-.}"` was not kept: the code sets that variable for hook commands only (`settings.go:117,160,179,207`), and nothing shows that a status line command gets it.
  No hub path argument is added.
- Support log: the Turn 3 bound-breaker 6 line no longer claims D1 names provenance, and a round 2 line replaces "_No rounds yet._".

## Self-check

Every lens returned a report.

- fix-claims MEDIUM, liveness engines built with `Dirs` lacking `Board` (`burlercli/wiring.go:202`): fixed. D6 requires every engine that reads or searches the known directories to be built with the hub's `Dirs`, and such a call site to tell them.
- fix-claims LOW, `CLAUDE_PROJECT_DIR` unverified for the status line: fixed; the command changes to the worktree's absolute path (see F12).
- fix-way-forward MEDIUM and fix-proposal MEDIUM 3, removing a logged-out record re-finds the directory: merged and fixed (see F9).
- fix-consistency MEDIUM 1, the told anchor list was not stated for the launch-time search and the resume check: fixed by the same D6 paragraph as the fix-claims MEDIUM, which names the engine that prepares, opens and resume-checks.
- fix-consistency MEDIUM 2 and fix-proposal MEDIUM 1, the scrub changes standalone runs and strands sessions that ran under a shell `CLAUDE_CONFIG_DIR`: merged and fixed.
  Scope Out names the exception, and D4 states the way forward: the operator sets the `env` module's value before the change lands, and an already-running server keeps its environment until it restarts.
- fix-consistency LOW 3, name-only verb siblings: fixed; D5's first paragraph and two D5 criteria name the directory.
- fix-consistency LOW 4, the D6 criterion contradicted `LocateSessions`: fixed; the criterion names the live-session transcript read.
- fix-consistency LOW 5, a stale support-log provenance line and no round entry: fixed in the support log.
- fix-consistency LOW 6, D3 step 5 did not write the raised counts, weights or listing: fixed.
- fix-proposal MEDIUM 2, the new resume-check bullet contradicted D2's empty-list bullet: fixed. The bullet applies with accounts listed, and with an empty list the check stats under the record's directory or the fallback and records nothing.
- fix-proposal LOW 4, "the caller" of the told ids was unnamed: fixed. D2 names the wiring that builds the run's engine set, collecting through each owning module's exported reader, so neither `shuttleengine` nor the claude engine reads reed's or orch's records.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
