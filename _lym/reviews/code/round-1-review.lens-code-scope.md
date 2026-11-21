```yaml
lens: code-scope
outcome: reported
```

Lens: code-scope. I traced each change in the diff to the board entry's goal points and the parent's added entry. Every change traces or is forced by a traced change, and every goal point is either built or settled without code.

**Decisions** (decision record alone: the goal points of the two board entries under `## Goal`, plus the one `Added after Discussion` entry)
- G1 (#160): check how triage derives identities, and whether `verify-command` can stand for a failure of the command.
- G2 (#160): keep the first run's failure output in the triage note.
- G3 (#160): keep a cwd-reset notice out of the failure identity and the note body.
- G4 (#158): run the `internal/reedcli` tests repeatedly to reproduce the failure and find the nondeterminism.
- G5 (#158): consider keeping the first run's failure output so a flaky identity can be diagnosed. This is the same outcome as G2.
- A1 (parent, 2026-11-21): waive red-first and keep spec commit 87f6be00.

**Changes → decision**
- C1. `VerifyGateNotes` stores `flakyFailure` (the `VerifyFailure` plus the first-run log): `addFlaky(first, firstLog)` and `firstLog := res.Log` (`verifygate.go:96-107`, `:484`, `:491`). Traces to G2 and G5.
- C2. `commandFailure` and `splitFlaky` (`triage.go:23-38`), and `triageWarnings` reporting the opaque failure apart (`triage.go:40-52`). Traces to G1.
- C3. `AppendIntegrationTriage(websterDir, flaky, commandLogs)` adds a second list for command failures (`summary.go:46-59`), and `Apply` passes it through `splitFlaky` (`verifygate.go:126-128`). Traces to G1.
- C4. `writeTriageFrictionNote` writes each failure's log and output tail, puts command failures in their own paragraph, and adds `writeFailureOutput` (`triage.go:54-106`). Traces to G2, G5 and G1.
- C5. `Flaky()` becomes the unexported `flakyFailures()` (`verifygate.go:111-116`). Forced by C1: the return type changes, and only tests and `Apply` call it, so keeping it exported would break `PATTERN-test-only-code`.
- C6. `doc.go:992-993`, `contracts/specs/webster-spec.md:72` and `:89-90`, and the file header in `triage.go:2`. Forced as docs C1 to C4 make stale. The spec hunk is the commit A1 keeps: the revert eae524ac and the reapply bf3009af cancel out, so the net diff keeps 87f6be00 as A1 says.
- C7. Test changes in `summary_test.go`, `triage_test.go` and `verifygate_test.go`. Forced by C1 to C5. The "a failing package" row's `tmuxkit: a stray entry` line also serves G4 and G5 as evidence of the kept output.

**Decision → changes**
- G1: built by C2, C3 and C4. The parser still emits `verify-command`; only the reporting treats it as a command failure (`runplan/verifyparse.go:118`).
- G2 and G5: built by C1 and C4.
- G3: no change, and none needed. No lym code writes "Shell cwd was reset" (internal grep found nothing). The verify log the tail comes from (`verifyparse.go:118`, `outputTail(lines)`) holds only the verify command's output. That command runs as lym's own subprocess, not through an agent's shell tool, so the harness notice cannot enter an identity or the note body. `_lym/landing/summary.md` ("No filter for the 'Shell cwd was reset to …' line") names this outcome and the reason. No finding.
- G4: no code change. The summary reports the investigation:
  - a gate run of `./internal/reedcli` passes on this base;
  - the cause is tmuxkit's stray `/tmp/.git` check, already narrowed by `5d8724d0`. I confirmed that commit exists, is an ancestor of the merge base, and does what the summary says.

  The entry asks for the tests to be run "repeatedly", and the summary cites one passing run. The cause was found from the log instead, which is what the repeated runs were meant to reach. Nothing the diff could change follows from this. No finding.
- A1: honoured by C6 (net spec diff = 87f6be00) and the summary's red-first note.

**Findings:** none. No unrequested scope, and no goal point is left unbuilt by a change it needs.

Tool calls:
1. `git show --stat 5d8724d0` and an ancestor check against `13cebe3d`, to verify the commit the summary cites to settle G4 (the context had only the summary's claim).
2. A grep of the hub for `webster-verify-triage*.md` notes holding "Shell cwd was reset" or `verify-command`, to check G3's premise against the original note. No such note exists locally.
