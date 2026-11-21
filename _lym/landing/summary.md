# Burler fix: tell a rerun fixer why; let forks push between own repos

Closes two defects from one burler fix round (issues #176 and #177).

## Fixer reruns with no reason (#176)

When the fix producer set a round's fixer report aside before the round finished, the next fixer over the same review had no way to know why it ran.
It happened on a failure exit, which archives the report alone so a resume skips the reviewer.
It also happened on the pre-attempt archive, when a died or timed-out first attempt had already written a report.
The report then sat under a timestamped name, the round looked unfinished, and the fixer had to read every card to tell a misnamed or rejected report from new work.

- `burlerengine.RunOpts` gains `Rerun`, a `FixRerun` holding a reason and the archived report's path.
  `fixWork` appends a paragraph when it is set.
  The paragraph says the fix runs again, not for new work, gives the reason, and names the archived report.
  It tells the fixer to keep each change that holds and to carry over each disposition that still applies.
- `BurlerFixProducer` keeps the reason in the round's snapshot record (`fixer_rerun`) under the accepted review's digest:
  - a failure exit records `the round halted: <error>`;
  - a cancelled exit records the cancellation;
  - the pre-attempt archive records the outcome of the attempt that left the report.
  The next call reads the record and passes it as `Rerun`.
  A record under another review's digest is never passed, so a fresh review is never described as a rerun.
- A retry whose died first attempt left no report is told the outcome alone.
- `archiveStaleOutput` is the single-file form of `archiveStaleOutputs` and returns the archived path, so the producer can name it.

## Fork git in its own scratch repositories (#177)

27498365 already let a fork run git inside a repository it created, but every `git push` still counted as mutating.
A fork that pushed from a scratch work repository to a scratch bare repository halted the round as a hard failure.
The halt's way forward said to revert what the fork wrote when nothing in the target had changed.

- A push now passes when its destination leads into a repository the fork created.
  That is a remote the fork pointed there with `git remote add`, `git remote set-url` or `git clone`, or a destination written as a path into one.
- `git clone` now records the repository it creates, at its directory operand or the name git derives from the source.
  Its origin, or the remote `--origin` names, leads in only when the source is a fork-created repository.
- A push still counts when it has:
  - no destination, or `--repo`;
  - a URL or a bare remote name the fork never pointed;
  - a remote renamed, or re-pointed outside;
  - a preceding `git config` or `-c` that names a `remote.`, `url.` or `branch.` key.
- `forkViolation` returns a `forkViolationError` that knows whether any fork wrote a file or ran a state-changing git invocation outside its own repositories.
  `auditFixRound`'s way forward now has three forms:
  - when the audit found no target change, it says so and its way forward is to resume;
  - when the violation was a push from a fork-created repository, it says to check that the push's destination is none of the target's remotes, then resume;
  - otherwise it keeps the old instruction to inspect and revert.
- `firstMutatingGitCommand` became `mutatingGitCommands`, which returns every violating command with the fork-created repository it pushed from.
  The way forward needs every violation, not only the first.

## Design decisions

- The rerun reason is persisted in the round snapshot record rather than re-derived from archived siblings.
  Only the producer knows why it set a report aside, and the digest key keeps a record from leaking into a later review of the same round.
- The issue offered accepting the archived report instead of rerunning.
  That is not taken: a report set aside by a failure exit, a fork violation for one, may be wrong.
  A report left by a died session never passed the fixer gate.
- Every push destination that is a bare word must be a remote the fork pointed.
  Treating an unknown bare word as a path would let a push to a remote set outside the transcript's view pass.

## Known gaps

- The audit stays lexical.
  A remote set by editing `.git/config` from the shell, or a `GIT_CONFIG_*` environment variable, is not seen.
  A push to such a remote passes only if its destination is also written as a path into a fork-created repository.
- A clone's relative source is resolved against the clone command's directory, while git resolves a remote's relative URL at push time.
  A push from a subdirectory of the clone can therefore reach elsewhere than the audit assumes.

## Tests

- `TestFirstMutatingGitCommandScopesToForkRepositories` gains rows for the issue's scratch push probe, a path push, clone origins, `--origin`, and each push form that still counts.
- `TestAuditClusterRound` asserts each fix-round violation's way forward:
  - the revert text for a write;
  - the no-change text for a named fork spawn;
  - the push text for a push from a fork-made repository;
  - the revert text when a push and a target commit share one command, or when a nested spawn sits beside a fork's target commit.
- `TestFixWork_Rerun` pins the rerun paragraph with and without an archived report, and its absence on a first fix.
  `TestEngine_Fix_TableShapeAndFiles` pins that `Engine.Fix` passes `RunOpts.Rerun` into the fix step.
- `TestBurlerFixProducer_Call_TellsARerunFixerWhy` covers four cases:
  - the reason and report after a halted call;
  - the same after a died attempt that wrote its report;
  - the reason alone after a died attempt that left none;
  - nothing for a record under another review's digest.

## Merge danger

**Door:** two-way
**Blast radius:** burler
