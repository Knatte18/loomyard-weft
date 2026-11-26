---
round: 2
focus:
  - "internal/websterengine/findingtext.go (recordsSideStatus, recordsRestoreStep), runlevel.go (Run's spawn) and internal/webstercli/sync.go (fabricSync commits the whole _lym pathspec at batch boundaries): which commit is a records-side path judged unchanged against, and can a Master write or deletion under _lym, committed by a sync before accept-audit runs, still read as unchanged? Check a state with no recorded baseline too."
  - "internal/websterengine/verifygate.go (NewVerifyGate, newVerifyGate, pass, recordGatePass): when the gate fails, fixers commit, the run stops and a restarted gate passes, which fixed head and base are persisted? Check also a restart that fails again before passing."
  - "internal/websterengine/auditledger.go (reconcileAcceptHead, AcceptPendingAudit): does the accepted commit range match the range the verify gate's commit check (fixCommitRejection in gitwrap.go) vetted, including commits between the last batch head and the gate's pre-fix head? Do the doc comments on these, State.FixedHead and doc.go state that range, and does a git probe error surface as itself or as a head refusal?"
  - "State fields in internal/websterengine that the run records for audit acceptance (MasterNotePath, PreFixHead, FixedHead and any baseline or base field beside them): is each cleared on reset (internal/webstercli/reset.go) and on a fresh State, and does a test pin each save, read and clear?"
  - "contracts/stencils/webster/webster-template-master.md (the write rules near lines 308 to 321) and contracts/orchrole/orch-role.md (the table row on Master's edits, near line 504): do all statements of Master's write surface agree, and does the stencil's pointer to the friction note name the section that renders it?"
  - "internal/webstercli/acceptaudit.go (acceptAuditCmd's long help): compared with the review base, is any statement dropped that the board entry did not ask to change, and does the text describe the behaviour the code has?"
---

The entries cover the places where round 1 found defects, as questions to settle against the current diff.
