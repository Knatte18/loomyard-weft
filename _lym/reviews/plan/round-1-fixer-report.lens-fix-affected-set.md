```yaml
lens: fix-affected-set
outcome: reported
```

I left these findings unchecked: F9, F12 and F13 (each concerns a single site, so there was no set to rerun), and card 1's remaining `Extends` entry under F7, because I ran out of tool calls before reading its Intent.

**MEDIUM: card 15's help text misdescribes hand-added Claude strands after the F1 fix**
- Changed text: `_lym/plan/15-creel-post-interrupt.md`, Intent, the sentence that rewrites `postCmd`'s Long and `Command`'s Long.
- Problem: it still says "a post reaches a Claude strand in full at its next tool call or turn end … a strand whose engine has no hooks by the typed notice or reminder".
- Under the parent's ruling, which the fix carried into cards 8 and 9, a Claude strand added by hand has no provider. It is therefore a pane-mail strand and gets the notice and reminder, not hooks.
- Evidence: card 8 now says "A strand with no provider, one added with `lym reed add`, has an unknown engine, so it is a pane-mail strand". Card 9 now says `TestSmokeCreelClaudeStrand`'s hand-added claude strand "stays a pane-mail strand".
- Related LOW: the CreelHooks refusal in the same card says "when its engine has no hooks". Under the ruling it should also cover a strand whose engine is unknown.

**LOW: F2's set of stale reminder texts is wider than the two orchengine sites the fix added to card 17**
- These texts describe the reminder flow, and no card edits them:
  - `internal/creelcli/cli.go:475-476`: `forwardCmd` Long says the receiver "gets a reminder".
  - `internal/creelcli/cli.go:720-721`: `holdCmd` Long says "only its mail is reminded".
  - `internal/creelengine/verbs.go:600`: `Store.Hold`'s doc comment says "only its mail is reminded".
- All three are stale for a hook strand, whose mail now arrives through its hooks. A grep of the plan finds no `forwardCmd`, `holdCmd`, `Store.Hold` or `Store.Forward` in any card.
- Excluded from this finding:
  - `internal/orchcli/creelseam.go:159` describes reminder journal records, which still exist.
  - `internal/creelcli/cli.go:642` and `:672` describe reminder events in the log, which pane-mail strands still produce.
  - The `docs/research` reports are measurement records.

**Checked with nothing found:**
- **F6:** the set of `interruptTarget` outcomes in `verbs.go`. All four are now wrapped or answered in card 15: the hold refusal, the pause-lifted refusal, the unreadable-pause refusal, and the after-key append failure.
- **F7:** apart from card 1's `reedengine/doc.go`, no card still has an `Extends` section.

Tool calls:
1. Grepped the repo for "remind" outside creelengine and `_lym`, to rerun F2's search, which my context lacked.
2. Grepped those hits in creelseam.go, status.go, cli.go and sandbox-hub.md, and grepped the plan for those files, to see which hits are stale and uncovered.
3. Read `forwardCmd` and `holdCmd`'s Long in cli.go and grepped the plan for them, to confirm the text and that no card covers it.
4. Grepped the plan for `Store.Hold` and `Store.Forward`, grepped creelengine for the reminder phrasing, and grepped the plan for other "Claude strand" claims. This checked creelengine's twin of item 3 and F1's set of sibling texts.
5. Grepped the plan for `Extends` sections, to rerun F7's set.
