```yaml
lens: code-correctness
outcome: reported
```

I reviewed the code-correctness lens: for each function and branch the diff adds or changes, I ran one concrete case through the code.

**Function and branch list:**
- `Call`, converting a `sessionStartHalt` to a halt: a non-transient start error becomes a halt whose reason names `lym loom resume`. Correct.
- `Call` → `endSession` when the context is cancelled: the session is left running. Correct.
- `Call` → `endSession` on a transient error: the session is stopped, then restarted by the re-step. Wrong (finding 5).
- `closeRecordedPullRequest`, record with a live strand: stop, then drop the record. A failed stop keeps the record. Correct.
- `openPullRequest`, closed pull request with no continue flag: starts a session, halts, stops it. Wasteful (finding 6).
- `openPullRequest`, merged pull request: no session (`keepsSession` false). Correct.
- `openPullRequest`, review already ended at the gated head: no session. Correct.
- `ensureSession`, recorded strand live: reused. Correct.
- `ensureSession`, strand dead: resumed through `ResumeSessionID`. Correct.
- `ensureSession`, `SessionLive` fails: error, re-stepped. Correct.
- `ensureSession`, record write fails after the start: the strand is stopped and the error returned. Correct.
- `ensureSession`, `RegisterSession` fails: logged only. Correct as documented.
- `holdWhileCovered`, covered then uncovered with a live strand: stop once, resume once. Correct.
- `holdWhileCovered`, stop fails: error. Correct.
- `wake` → `sessionStopWanted` true and `judge` ends the call: stopped, verdict given. Correct.
- `wake` → `sessionStopWanted` true and `judge` keeps waiting: the session is stopped and restarted on every poll. Wrong (finding 1).
- `wake` → `haltOnEndedSession`, strand dead: halts. Correct.
- `wake` → `haltOnEndedSession`, liveness read fails: one Warn, wait goes on. Correct.
- `gateHead` pass: push, then record `GatedHead`. Correct.
- `gateHead` crash: halts and mails nothing. Correct.
- `gateHead` failure with `MailedHead` equal to the head: skipped. Correct.
- `gateHead` failure where the mail fails or no strand is recorded: `MailedHead` is still recorded. Wrong (finding 2).
- `gateHead` failure inside a wait whose review has ended: the row ends Done. Wrong (finding 3).
- `prContinueVerb`, strand `""`: admitted. Correct.
- `prContinueVerb`, `lymhub:task:darn`: refused before any read. Correct.
- `prContinueVerb`, unparsable strand: refused. Correct.
- `prContinueVerb`, `lymhub:other-task:pr-reviewer`: admitted. Wrong against the code's own doc (finding 7).
- `prContinueVerb`, continue flag set and a mailed head recorded: the mailed head is cleared and the continue flag stays set. As written, but it couples a gate retry to ending the review (finding 4).
- `RenderPRReviewPrompt` / `runMaterial`, no material on disk: `- none`. Correct.
- `MailAgent` wiring, client construction fails: the error is returned, then logged by `mailFailure`. Correct.

**Findings:**

1. MEDIUM, design, `internal/landingshed/prreview.go:601-616`. The session is stopped and restarted at every poll whenever `sessionStopWanted` holds but `judge` keeps waiting.
   - **Cases:**
     - An approval at the gated head while the tree is dirty, or while the local head is past the gated head (for example a mailed failing head).
     - An approval or the continue flag at the gated head while `textStale` is true.
   - **Effect:** once a minute the operator's agent is killed mid-turn and restarted by session id.
   - **Evidence:** `sessionStopWanted` (prreview.go:652-660) requires only `HeadSHA == GatedHead` for an approval, and only clean plus gated for the continue flag. `judge` (prreview.go:847-858) also requires `!staleText`, `HeadSHA == head` and a clean tree.
   - **Fix:** make `sessionStopWanted` test the same conditions `judge` ends on.
2. MEDIUM, design, `internal/landingshed/prreview.go:715-722, 727-735`. A gate failure whose mail fails, or that has no recorded strand to go to, still records `MailedHead`.
   - **Effect:** the head is never gated again, the agent and the operator are never told, and the wait sits silent with only a log Warn.
   - **Evidence:** `mailFailure` swallows the post error, then `updateRecord` sets `MailedHead = head` unconditionally.
3. MEDIUM, design, `internal/landingshed/prreview.go:705-722` with `judge` at 826-827 and 838-841. In a wait on a review that has already ended (`keepsSession` false), a new commit is gated. When that gate fails, `gateHead` mails and returns nil.
   - **Effect:** `judge` then returns Done for a clean tree, whatever the head, so a failed and unpushed head passes PR-Review to Finalize. Before this change, a gate failure halted here.
   - The mail also goes to the strand of a session that was already stopped.
4. MEDIUM, design, `internal/landingshed/prreview.go:678` (`gateRetryHint`) and `internal/loomcli/prcontinue.go:70-73`. The mailed retry route, `lym loom pr-continue`, also sets the continue flag.
   - **Effect:** an agent that follows the mail to retry a failure from outside the change ends the review as soon as the re-gate passes, without the operator's word.
   - The flag also survives a re-gate that fails again, so a later commit that passes the gate ends the review automatically.
5. LOW, design, `internal/landingshed/prreview.go:92` and `prsession.go` `endSession`. Every error the driver re-steps stops the session first.
   - **Examples:** a transient GitHub failure on the entry status read (`githubFailure`, prreview.go:244), or a failed record commit in `updateRecord`.
   - **Effect:** the operator's live agent is killed mid-exchange, then restarted by the re-step.
6. LOW, design, `internal/landingshed/prreview.go:265-272` with `judge` at 835. A re-entry on a closed pull request with no continue flag starts a session only for `judge` to halt and `endSession` to stop it at once. Every `lym loom resume` repeats this.
7. LOW, design, `internal/loomcli/prcontinue.go` `prContinueAdmitsStrand`. Any strand whose role is `pr-reviewer` is admitted, whatever its slug and whatever `rec.StrandName` says.
   - **Contradiction:** its doc and the refusal text say "the run's PR-Review session".
   - **Example:** `lymhub:other-task:pr-reviewer` and the slugless `lymhub:pr-reviewer` both pass.

**Tool calls:**
- Read `transientFailure` (ctx.go:41) and `halt` (stuck.go:55), to confirm the start-error classification and what `endSession` receives for a halt. The manifest did not hold either.
