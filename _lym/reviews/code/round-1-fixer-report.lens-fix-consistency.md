```yaml
lens: fix-consistency
outcome: reported
```

Two stale siblings from the change set:

**MEDIUM** — `PATTERN.md:66`, the index line for `PATTERN-heddle-instructions`.
- It reads "the Bolt files written only by `lym heddle` verbs and heddle's Bolt replayer".
- F4 changed `pattern/PATTERN-heddle-instructions.md:8-9` to say the replayer writes `heddle.json` only, and the verbs alone write `heddle-lym-override.json`.
- The index line is the entry agents match against and it still states the old rule. It is the twin of the edited bullet, and the fix left it unchanged.
- Fix: reword it to "`heddle.json` written only by `lym heddle` verbs and heddle's Bolt replayer, the override file only by the verbs".

**LOW** — `internal/heddlecli/doc.go:6`, the `list` description.
- F2 added a `warnings` field to the `list` output and an "ignored override" sentence to the verb's `Long` text in `verbs.go`.
- The package doc's `list` paragraph does not mention either. It stays true but is incomplete next to the help text.
- Fix: add "an override file that does not load is reported in warnings and ignored".

Checked and consistent, no finding:
- `verbs.go` set/remove `Long` text, `heddlecli/doc.go:9`, `refuseShipped` and the `Set`/`Remove` doc comments all agree on the new-entry-only refusal.
- The `Replayer.Claims` doc, `heddleengine/doc.go` and the new `Claims` test row agree that the override file is unclaimed.
- `claudeengine/doc.go:224` now matches `Passages`.
- The F8 help-text revert restores the pre-change line.

Further tool calls:
1. `grep` over `PATTERN.md`, `docs/overview.md`, both `doc.go` files and the heddle pattern file, to find sibling wording of the changed claims. My context held the edited files but not the index docs.
2. `grep` and `sed` over `PATTERN.md`, `docs/overview.md` and the first lines of `heddlecli/doc.go`, to read the index line and the `list` paragraph. My context held neither.
