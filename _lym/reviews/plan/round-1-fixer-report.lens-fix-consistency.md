```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the sibling cards, decisions and cross-references they touch.

- **LOW**: card 5's new `push_skipped` envelope field (`commitRecordsVerb`, and `addWeftVerbs`' `lym fabric push`) is not added to any envelope doc. Card 5 extends only `internal/fabricengine/doc.go`, but `internal/fabriccli/doc.go:31` already lists `lym fabric push`'s envelope fields (`local_skipped`), so a twin line for `push_skipped` was missed. Evidence: card 5's Extends block and its doc-update bullet name no fabriccli or loomcli doc; `internal/fabriccli/doc.go:31`.
- **LOW**: card 1's line in the overview Card Index still reads `retry "missing necessary objects" and return ErrLeaseRejected from a leased delete`. After the fix, card 1 only adds caller-named extra reasons (`PushWithRetryReasons`), and the reason itself is passed by card 6. Card 1's commit title was changed to `gitrepo retries caller-named refusals…`, but its index line was not. Evidence: `_lym/plan/00-overview.md:14` against card 1's Intent item 1 and Commit.

Every other changed cross-reference agrees:
- Card 12's closed set, now reached by card 4's records-branch refusal.
- Card 12's transient list.
- The `removeFields` edits on cards 6, 7 and 13 (only card 13 re-signs it).
- Card 6's Uses of `PushWithRetryReasons`.
- The overview's D3 decision, which matches card 13's new `preflightRunOwner` site.
- The tier split between card 10's two tests.

Tool calls:
- One grep over the plan cards and the fabriccli and loomcli doc files. It checked the changed symbols' other mentions and the envelope docs, which my context held only for the cards the fixer read.
