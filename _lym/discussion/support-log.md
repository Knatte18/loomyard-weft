# Support log — compact-inject-draft

## Interview

A darn run holds no interview; the writer read the board entry, the orch's transcript, events file and run settings, the creel journal and the code, and made the calls in the decision record.
The parent `ly:orch` added issue #899 to the scope by message.

## Rejected alternatives

- Racing the `asyncRewake` Stop waiter, as the board entry suspected: the orch session ran settings from before the deploy, with no waiter.
- Lengthening the submit window for a compact send: compactions run past any fixed window, and the failure came inside it.
- Having creel clear and retype an unconfirmed command inject: the command had landed, so a retype would compact twice.
- Setting `PaneMail` from `CreelHooks` in `internal/creelcli`: it keeps a field that only restates `CreelHooks`, and a later caller could set the two apart again.
- Ringing a strand without `CreelHooks`: it has no waiter to ring, and the bell line tells the session that creel hands mail over at its turn end, which no Stop hook of its does.

## Review rounds

_No rounds yet._

## Question ledger

- What made #107 read unconfirmed? Self-pick: the operator's draft read as foreign box text after the submission, per the transcript and the journal.
- Did the cycle hang? Self-pick: no; the watcher waited for the compaction boundary and the later steps went in at 13:45.
- Which strands keep the pane path? Self-pick: every strand not marked `CreelHooks`.
- Is #899 in scope? Resolved by the parent: yes, and the change closes both issues.
