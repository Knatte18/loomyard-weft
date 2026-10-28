# A landed cycle command reads as injected; mail wakes every idle strand

Two faults in creel's delivery into an idle orch strand are fixed.
A `/compact` that the session took could be journaled `unconfirmed`.
Mail to a Claude strand launched without creel's hooks could sit unread with no ring, bell or reminder.

Closes #897
Closes #899

## What happened in #897

The orch's records show that the watcher's `/compact` did land.
The Claude transcript records it submitted at 13:44:06.083, 150 ms after creel's `sending` record for inject #107.
The compaction ran until 13:44:43.
The orch session ran settings from 2026-10-27 with no `hook-wait` entry, so the suspected `asyncRewake` race could not have happened.
While the compaction ran, the operator typed into the input box.
Shuttle's paste confirmation, still waiting for the compaction's session start, read that draft as foreign box text and failed with `ErrForeignBoxText`.
Creel maps that error to `ErrNotLanded` and journaled #107 `unconfirmed`.
The later `draft` stall was the operator's own text, and the cycle went on: color, plugins and the resume line went in at 13:45:15–19.

## What changes

- Shuttle (`confirmPasted`): a send that confirms on a session start, which covers `/compact` and `/clear`, counts as submitted once its box reads empty after the paste.
  From then on, any box text and any unreadable box are read again with no key sent, and the end of the submit window confirms the send, as an empty box already did.
  A send that never read its box empty keeps today's rules: Enter for its own text, `ErrSentTextInDraft` for its text beside a draft, and `ErrForeignBoxText` for other text.
- Creel: `Strand.PaneMail` and `creelcli`'s `hasHookDelivery` are gone.
  A strand marked `CreelHooks` is rung or belled, and every other strand gets the notice and the reminder, whatever its provider.
  Before this, a strand whose provider renders hook delivery but whose session lacked it fell into neither branch.
  Such strands are a Claude session launched before `creel-hook-post`'s deploy, a `lym shuttle open` session and a hand-added Claude strand.
  For the orch in #899 that meant mail #106 from `ly:ORCHWATCH` sat unread about 90 seconds.
- Docs: `shuttleengine`, `creelengine` and `docs/overview.md` state both rules.

## Design decisions to review

- After the box has read empty once, shuttle accepts a landing send without reading what comes next.
  The evidence standard is the same one every other paste already lands on.
  It never sends a key afterwards, so an operator's draft is never submitted or cleared.
- Creel removes a field rather than deriving it from `CreelHooks` in `creelcli`, so no caller can set the two apart again.
  A strand with an older tool-boundary claim hook may still claim mail at a tool call.
  The pane path reserves a notice before it types it, so each message is delivered once, and agent mail stays reminder-only.
- Out of scope: retyping a command inject creel journaled `unconfirmed`, since the one in #897 had landed and a retype would compact twice.
  The orch watcher's handling of `unconfirmed` also stays unchanged: it already waits for the compaction boundary.

## To check by hand

- Run a live `/compact` cycle on the orch while typing into its box during the compaction, and check that `lym creel log` shows the compact inject `injected`.
- Post to an orch strand launched before the hook delivery while it is idle, and check that the notice is typed.
- During this work, `TestCreelCLI`'s EXTERNAL row failed twice under a loaded machine.
  It then passed alone, as the whole test three times, as the whole package twice and in the final scoped `--upto 2` run.
  Its assertions read nothing the change touched, so check whether it flakes under load.
