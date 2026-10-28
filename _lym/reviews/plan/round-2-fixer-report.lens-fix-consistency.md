```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the changed text in cards 7, 8, 15 and 18 still agrees with the parts of the plan it touches.

1. LOW. Card 15, the new Intent sentence "an error `interruptTarget` returns together with false that is none of the three, a key the delivery failed to send, is returned as `interruptTarget` words it, with nothing stored".
   Summary: the Intent gained a new outcome, but the Tests paragraph has no row for it.
   Evidence: every other Intent branch of `post --interrupt` has a twin row in `TestStore_Mail` (hold, lifted pause, unreadable pause, no running turn, the two store failures after the key). The fake already supports a failed key through `SetInterruptError`, but no row uses it to check that the error comes back unwrapped and nothing is stored.
2. LOW. Card 15, the new Intent sentence that makes `interruptTarget`'s refusals "a typed error carrying the refusal's kind, whose message is today's, so `Interrupt` and `inject --interrupt` answer as they do today".
   Summary: the change now reaches the refusal paths of `Interrupt` and `inject --interrupt`, but no test of those verbs is named to pin their unchanged messages.
   Evidence: `interruptTarget` is called from `verbs.go:426` (inject) and `verbs.go:529` (`Interrupt`). The **Edit:** list names only `TestStore_Mail`, not `TestStore_InjectAndHold` (`verbs_test.go:383`). The claim holds only if an existing row already asserts the refusal text, and the card does not say which row that is.
3. LOW. Card 18, the line "in none of them is a reminder or notice line typed into the pane", left as it was.
   Summary: the F2 fix moved the bell's "never typed" check from the pane to the events file, but this sibling "never typed" check still reads the pane.
   Evidence: the review's F2 reasoning applies here too. `capture-pane -p` has no `-J` and no scrollback, so a long typed line wraps or scrolls out, and a negative pane check passes without proving anything. The changed idle-wake and interrupt-and-bell sentences now assert typed lines through `UserPromptSubmit` payloads.

Further tool calls:
- grep across the plan cards for the changed symbols and phrases: to find twins of the changed text, which my context did not show for the cards it never read.
- grep of the overview and cards 1, 9, 11 and 17 for hand-added strands and bell-in-pane wording: to check whether the twins of the F5 and F2 changes agree.
- grep for the cards and tests that touch interrupt symbols, plus the `TestStore_` functions in `verbs_test.go`: to find which tests reach the refusal paths now under Edit.
- grep of cards 3, 4 and 6 for interrupt symbols: to rule out a second card that edits `interruptTarget` (none does).
