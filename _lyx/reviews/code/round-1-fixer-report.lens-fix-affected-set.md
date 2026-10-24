```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The two fixed sites are consistent. In `decision.go`, line 22 and line 538 already use the plural ("each entry's heading", "each entry's title").

One sibling the reviewer didn't name still reads singular:
- **LOW**, `docs/overview.md:518`, in the `--amends-plan` bullet. The text reads "the call also appends the entry's title to the plan amendment record, committed with the entry".
  - The line sits two lines below the `--from` paragraph at 516, which says a batch records many entries in one commit.
  - It is the same defect as F1 and F2, so you'd likely want it fixed alongside them.

Tool calls (1 of 6):
1. `grep` for singular "entry's title/heading" phrasings across `.go`, `.md` and `.yaml` files. My context held only the two named sites, so I needed the other sites that describe the same behaviour.
