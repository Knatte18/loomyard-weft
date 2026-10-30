```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: checking that each changed piece of text agrees with the cards, decisions and cross-references around it, and that no change is missing its twin elsewhere.

1. [LOW] Card 11's `README.md` sentence (`…lands the task through a GitHub pull request when the resolver says so, since the session the operator discusses it with arrives only in card 14`).
   The fixer moved the "discusses with an agent" part of the reviewer's wording to card 14, but card 14 never adds it.
   Evidence: card 14's **Edit:** list has `internal/landingshed/doc.go` but neither `README.md` nor `docs/overview.md`, and its Intent says only "The landingshed doc's PR-Review paragraph describes the session."
   Once the plan is built, the README's Landing bullet still says nothing about the agent session.

2. [LOW] Card 13, `Call ends a yes with wait instead of Done.` (line 44).
   Card 11's new step 6 says the kept-`ClosedForContinue` path's Done is one "which card 13 turns into the wait whose closed-and-`ClosedForContinue` verdict ends Done".
   Card 13 has no matching sentence about the kept-record path; its new test-row clause ("which `Call` keeps and hands to the wait") is its only mention.
   The generic sentence covers the path, but a reader of card 13 alone does not learn that `Call` skips steps 7 and 8 for the kept record before it calls `wait`.

The other twins agree with the change set:
- **Card 9:** pr-continue on a `ClosedForContinue` record writes nothing new.
- **Card 15:** a `ClosedForContinue` record with a closed pull request lands locally, and `Continue` without `ClosedForContinue` halts.
- **Card 14:** `ensureSession` skips the kept record, and its test row was added.
- **Card 16:** the guard's envkit exemption matches the envkit stub in card 11.
- **Overview:** the new Shared Decision agrees with card 13's step 4 and its new test row.
- **Card 8:** the allowlist entry is in the card where `ParseOwnerRepo` is first imported, so card 11 does not add it twice.

Tool calls:
- One grep of the plan cards other than 11 and 13 for `ClosedForContinue`/afresh, for `pullrequest` imports in cards 9, 15 and 17, and for `gate.check`/poll-bound mentions. My context did not show cards 7, 9, 15 or 17, or any of the cards after the edits.
