# Webster stability: gate shells, recovery writes, partitioned members

Six faults outside the audit stopped or misled Webster runs.
This change closes them.

## What changes

- **Gated turn end** (`internal/shuttleengine`): a gated run's waiting turn end with any background task outstanding, a shell included, no longer counts as an arrival while Master's output files exist.
  The run waits for the task's completion, bounded by the run deadline and the liveness check, as it already does for a fork.
  A run can no longer read `done` with a gate tier unfinished because a background gate shell was cut off.
  Shells a deadline, liveness failure or stop still ends over stay in `EndedShells`, and the warning, friction note and summary section say the shell's result was never read.
  Ungated runs keep today's rule.
- **Verify gate re-prompt** (`shuttleengine`, `internal/websterengine`): `GateEntry` takes an optional one-line re-prompt renderer.
  Webster's `verify` entry uses it, and both the re-prompt and the findings text Master reads open with the way forward: spawn the fixer fork, never fix, edit or commit itself.
  Other gated producers keep the default re-prompt.
- **Recovery attribution** (`websterengine`): every recovery strand's session is recorded on the batch and audited for write evidence.
  A second recovery's prompt lists the first recovery's uncommitted files under "Written by this run", and a reset may discard a tracked path a recovery of this run wrote.
  Contract-file evidence counts recovery writes beside fork writes, and its messages name the last writer as a fork or a recovery session.
- **`lyx webster recover-batch`** (`internal/webstercli`): exits non-zero with `ok: false` and the same digest fields whenever the returned batch is terminal and not `done`, so `stuck` and `dead` join `failed`.
  The message names the ladder's way forward.
- **Implementer body** (`contracts/stencils/webster/`): names the card gate, the batch gate, `lyx loom lint-comments` and `lyx quarry resolve` as always allowed, and says the fabric ban covers mutating `lyx fabric` verbs only.
  A `tmux`- or `llm`-tagged test no gate step names is compiled through `lyx gate test --tags <tag> <packages> -- -run '^$'` and never run.
- **Done-check findings** (`internal/planglyph`): `create-not-done`, `delete-not-done` and `rename-not-done` name the resolve key looked up, quarry's status, the unit status for `not_found`, and for `ambiguous` each candidate's file and build constraint.
- **Partitioned members** (`planglyph`): a member whose every declaration sits in files no single build environment selects together is one member.
  Edit, Delete, a Rename's new side, `Uses:` and the Create done-check all accept it, so a member declared once per build-constraint set or once per GOOS can pass.

## Design decisions

- **Classifier.** A single classifier over an `ambiguous` answer's candidates evaluates each candidate file with `go/build.Context.MatchFile`.
  The environments are every pinned GOOS/GOARCH pair, cgo on and off, and every assignment of the free tags the constraints mention.
  Free tags include `gc`, `go1.N` and `goexperiment.*`, so no tag is held true by the toolchain lyx was built with.
  More than four free tags, an unreadable file or an unparsable constraint leaves the answer `ambiguous`.
  quarry is unchanged.
- **Rename.** A Rename's old side stays Found-only, as for `multipart`, because the new declaration is derived from one signature.
  Its refusal names the way forward: a Delete of the old member plus one Create of the new handle on a card that edits each old file, or an Edit of each file's body.
- **Create.** `createFindings` still treats an ambiguous Create target, partitioned or not, as `create-already-exists`.
- **Genuine ambiguity.** A genuinely ambiguous Edit or Delete target stays blocked at Plan time, with each candidate's file and constraint named.
  The only ambiguity that can first surface at record-batch is one the batch's own Create introduced, which the new done-check detail tells apart from a missing symbol.
- **Deadline outcome.** A gated run held on a shell that never reports ends `OutcomeDone` at its deadline, because finalizing a negative verdict over a satisfied file contract is what `PATTERN-completion-signal` forbids.
  The gate runs once more at finalize.
- **Docs.** The module docs in `internal/shuttleengine`, `internal/websterengine`, `internal/webstercli` and `internal/planglyph`, and the refusal-spec and loom plan spec rows, change with the code.

## To check by hand

- A shell whose completion notification shuttle never recognizes now holds a gated run until its deadline, where it ended the run `done` at once before.
  Such a run ends `OutcomeDone` with the gate run once at finalize and the shell in `EndedShells`, its result unread.
- The re-prompt wording and the implementer stencil wording can only be judged by a live run; no `llm`-tier test covers them.
  Check that a `sonnet[medium]` Master follows the fixer-fork line on a verify failure, and that a recovery stops asking whether the gates are allowed.
- The verify gate's first attempt failed `TestCLIForm_EveryCommand` on the `recover-batch` `Long` help (2196 bytes against the 2000-byte cap); it is tightened in this change.
  `lyx gate test ./...` and `lyx gate test --tags integration ./...` were not run over the merged tree here.

Closes #505
Closes #509
Closes #517
Closes #522
Closes #511
Closes #500
Closes #507
