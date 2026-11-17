```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checking each bound, behaviour and fact the fix added against the code at HEAD.

Claims checked and found to hold:
- "The review row carries neither `NotifyParent` nor `decisions_escalated`" (D5). This holds: `internal/shedrecipe/entries_burler.go:130` builds `BurlerHalfDeps` with neither field, and only the fix row at `:208` sets them.
- "`_lym` is a link out of the worktree" (D3). This holds: `_lym` is a symbolic link to `review-rounds-converge-weft/_lym`.
- "Through `lymcwd` alone … from the seam cwd" (D3). This can be done: `lymcwd.ResolveWithin` resolves from a cwd and is meant for read-only verbs.
- The digests use the "heading and body as `discussionparser.AddedDecisionTexts` returns them", and the heading is "dated". Both hold: `internal/discussionparser/append.go:189` returns heading and body verbatim, and `:180` requires each heading to carry an author and a date.

Findings:

- LOW — D3 held bullet: "the block lists only entries the parent added through `lym loom decision add` since round N-1 began, so a `resolved` status always rests on an answer the parent gave in that span". The bound names the parent as the only writer of these entries, but the arbiter is also told to use `lym loom decision add`. Evidence: `internal/loomcli/arbiterprompt.go` names "decision add". The bound still holds if it reads "the parent or the arbiter", or "entries written through `lym loom decision add`".
- LOW — D6 added sentence: "the check yields to the code after it, so `softCapRule` accepts the round as today". When the round has an escalation record, the code after the check is the escalated-record re-escalation, not `softCapRule`. Evidence: `internal/shedadapters/bouncer.go:1361-1366`, where the `escalated || readErr != nil` branch comes before `softCapRule`. The sentence is right that the check yields, but "softCapRule accepts" overstates the case; "the round settles as today" would be exact.

Tool calls:
1. Read `internal/shedrecipe/entries_burler.go:70-80,125-135` and `ls -ld _lym`, to check the review row's deps and the `_lym` link claim. My context held only the grep line for the fix row.
2. Grepped `internal/loomcli` for who runs `decision add`, to check the claim that only the parent writes entries. My context held no caller list.
