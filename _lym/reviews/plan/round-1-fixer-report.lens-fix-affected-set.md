```yaml
lens: fix-affected-set
outcome: reported
```

I checked whether each fix covers every member of its finding's set. I reran the searches for F2, F3/F4, F7 and F8; F5, F6 and F9 are single sites with no set.

Unchecked: F1 (a decision-class finding, outside this fix), F5, F6, F9.

**LOW: card 7 "Env and entry" paragraph, which F8's fix leaves stale.**
F8's premise is that card 7 leaves comments untrue in the files it edits. The fix reworded the tapestry comment block and `TestApproveSeam_FailsToBuild`'s doc comment.
Another comment in an edited file stays untrue: `internal/shedrecipe/entries_bouncer.go:113-115` says approve_seam "resolves to env.ApprovePlan and nothing else".
That block is neither `bouncerEntry`'s doc comment (`:30`) nor the switch comment (`:139`), which are the two comments card 7 names.
Evidence: `entries_bouncer.go:113-115`. Card 7 says its doc comment and switch comment "name both", and it names no other comment in that file.

**Sets confirmed covered:**
- **F2:** `Board.Rename` has one caller outside its package, `internal/boardcli/cli.go:235`. Card 5 now covers its help and the skill's rename text at `plugins/ly/skills/board/SKILL.md:100-104`. The callers of RemoveTask, Prune and MergeTasks (`boardcli/cli.go:258,690,935`, `battencli/land.go:110`, `cleanupcli/segments.go:86`, `fronttask/hub.go:46`) need no change, because a failed removal now only logs.
- **F7:** the docs stating the front/on_land payload rule are `boardcli/doc.go:29` (card 5 now covers it), and `boardengine/store.go:457` and `boardcli/cli.go:511` (card 1 covers both).
- **F3/F4:** the `pastErr` and pastApproval-error rows and the "HEAD does not hold" row are gone from cards 8 and 9. Card 9's HEAD-not-held refusal stays and can be reached, since the store verb makes no commit first. Remaining mentions are only in the earlier lens-review files, which are records.

Tool calls:
1. grep for the approve_seam comments: my context lacked the other comments in entries_bouncer.go and entries_gate.go.
2. grep for the front/on_land rule docs: my context lacked the other sites stating that payload rule.
3. grep for pastErr and HEAD-not-held in the cards: to confirm the twins in cards 8 and 9 were removed.
4. grep for the callers of the Board write methods and the skill's rename text: my context lacked the caller set and SKILL.md.
