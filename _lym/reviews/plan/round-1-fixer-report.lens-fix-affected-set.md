```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I reran the searches behind F1, F2 and F4. F3 is not a set; F5–F7 each name a single site.

Findings left unchecked: none of the set-shaped ones. F3 and F5–F7 have no set to rerun.

- LOW — card 5, `TestBindings_StatusBarClicksSelectAndZoomByRangeType` bullet ("gives its odd session `status 3`"), which is part of F2's set of sites that pin the two-line bar.
  Summary: the fix covers `TestSmokeSelvage`'s assertion and its comment. Its sibling in the same set changes the value, but the card does not mention the text around that value.
  Evidence: `internal/reedengine/bindings_integration_test.go:338` has the comment "The session shows two status lines only because it is given them". Lines 343–344 call `set-option ... "status", "2"` and have the Fatalf message `"status 2 on %q"`. Card 5 changes only the value to `status 3`. It names no change to that comment or that error message, although it does name both for `TestSmokeSelvage`.
  Every other member is covered:
  - `windowsize.go:76` is the `status 2` pin.
  - `windowsize_test.go:46-51,65` are the doc comment, the testtiming reason and the `wantOptions` entry.
  - `smoke_selvage_keepalive_test.go:273,338` are now in card 5.
  - The Windows branch at line 295 maps any integer and stays correct.

F1: every plan site that named `run-shell -c` or the `-c` start directory has been changed:
- card 8, step 6, `tmuxQuote` and the test row
- card 9, the binding and `toolfloor`
- card 1 adds the word form

The only remaining mentions of `-c` are explanatory text, plus `CommandFlag` (the shell's `-c` flag), which is unrelated. No card other than 8 and 9 composed `run-shell`.

F4: the strand flag is read only by card 8 step 3 and pinned only by card 6's row, and both agree with the new definition. Card 3 rejects `{strand}` without `window: agents`, so `strand` true still implies an agent-row entry.

Tool calls:
- A repo-wide grep for status-2 and two-line-bar assertions: my context held only `smoke_selvage_keepalive_test.go:325-345`, not the full set.
- `smoke_selvage_keepalive_test.go:288-300` plus a plan grep for `run-shell -c`, `tmuxQuote` and `lym_button_dir`: to check the Windows status branch and every F1 site across the cards.
- A plan grep for sites that key on the spec's `strand` flag: my context had not searched cards other than 6 and 8 for that flag.
