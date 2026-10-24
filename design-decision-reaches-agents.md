# A parent decision reaches the agent that needs it: Burler decision findings and Merriam

`decision-reaches-agents` · Task · bug, burler, loom, webster, stencil, creel

## Part 1: Burler fix loop cannot route a decision-class finding (from `burler-decision-finding-route`)

Imported from [issue #853](https://github.com/Knatte18/loomyard/issues/853).

A Burler plan review raised a `decision`-class finding whose fix is to ask the parent to record a decision.
Round 1 raised the same finding and its fixer deferred it.
Round 2 raised it again, and the fixer could only defer it again.

Why this is abnormal: the fixer's prompt forbids any message about the review to another session, and the decision record is outside the fixer's write surface.
The fixer can neither ask for the decision nor record it, so every round re-raises and re-defers the finding.
Nothing in the loop routes a `decision` finding to the parent.

What to do: route `decision`-class findings outside the fix step, for example escalate them to the parent (`ly:orch`) from the Burler round driver, and stop counting them as fixable work for the fixer.

Provenance: task `batten-cancel-verb`, note `burler-plan-r2.md`.

## Part 2: Merriam refuses the decision notice (from `merriam-refuses-decision-notice`)

Imported from [issue #849](https://github.com/Knatte18/loomyard/issues/849).

Seen (2026-10-23, loomyard hub, run plan-card-rules at Webster): the orch ran `lyx loom decision add plan-card-rules --by parent` to tell the run to port main's reed separator fix. The verb's notice reached Merriam (`ly:plan-card-rules:webster`) as creel #157 from `ly:LOOM`: 'a new decision is in the decision record .../_lyx/discussion/decision-record.md ...; re-read the record's Added after Discussion entries and follow them'. Merriam refused it: it trusts only `ly:orch` as its parent, and its stencil forbids it to read decision-record.md. So a parent decision never reaches a running Webster, although the orch role says the verb tells each live agent of the run.

Workaround: the orch messaged Merriam directly, with the decision's content spelled out.

Expected: a decision recorded while Webster runs reaches Merriam in a form it acts on, for example with the decision's text inlined in the notice and a sender Merriam's stencil accepts, or Merriam's stencil admits the decision notice and the record's Added after Discussion section.

## Issue comment, 2026-10-23

Cause, from the code: `lyx loom decision add` posts the notice through creel (`postDecisionNotice`, internal/loomcli/decision.go), signed with the program name `LOOM` (`loomProgram`, internal/loomcli/wiring.go). Creel types a post into the receiver's pane as a user turn opening `[creel] #<id> from ly:LOOM:`, so Merriam reads a sender that is not its parent `ly:orch` and refuses it. A second refusal follows from the text itself: it asks Merriam to read decision-record.md, which the Master stencil does not list among its reads.

The same happened in a second run (batten-cancel-verb, creel #158).

A fix needs both parts: a sender Merriam accepts as its parent (or a Master stencil that admits a `LOOM` decision notice), and the decision's text inlined in the notice, with the instruction to hand it to the verify fixer fork, since Merriam edits no source.