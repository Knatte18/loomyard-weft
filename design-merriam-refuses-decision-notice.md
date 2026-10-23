# Decision notice from lyx loom decision add is refused by Webster's Merriam

`merriam-refuses-decision-notice` · Note · bug, webster, loom, stencil, creel

Imported from [issue #849](https://github.com/Knatte18/loomyard/issues/849).

Seen (2026-10-23, loomyard hub, run plan-card-rules at Webster): the orch ran `lyx loom decision add plan-card-rules --by parent` to tell the run to port main's reed separator fix. The verb's notice reached Merriam (`ly:plan-card-rules:webster`) as creel #157 from `ly:LOOM`: 'a new decision is in the decision record .../_lyx/discussion/decision-record.md ...; re-read the record's Added after Discussion entries and follow them'. Merriam refused it: it trusts only `ly:orch` as its parent, and its stencil forbids it to read decision-record.md. So a parent decision never reaches a running Webster, although the orch role says the verb tells each live agent of the run.

Workaround: the orch messaged Merriam directly, with the decision's content spelled out.

Expected: a decision recorded while Webster runs reaches Merriam in a form it acts on, for example with the decision's text inlined in the notice and a sender Merriam's stencil accepts, or Merriam's stencil admits the decision notice and the record's Added after Discussion section.

## Issue comment, 2026-10-23

Cause, from the code: `lyx loom decision add` posts the notice through creel (`postDecisionNotice`, internal/loomcli/decision.go), signed with the program name `LOOM` (`loomProgram`, internal/loomcli/wiring.go). Creel types a post into the receiver's pane as a user turn opening `[creel] #<id> from ly:LOOM:`, so Merriam reads a sender that is not its parent `ly:orch` and refuses it. A second refusal follows from the text itself: it asks Merriam to read decision-record.md, which the Master stencil does not list among its reads.

The same happened in a second run (batten-cancel-verb, creel #158).

A fix needs both parts: a sender Merriam accepts as its parent (or a Master stencil that admits a `LOOM` decision notice), and the decision's text inlined in the notice, with the instruction to hand it to the verify fixer fork, since Merriam edits no source.