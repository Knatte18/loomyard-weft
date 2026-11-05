```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: checked the `rename-not-done` and exhausted-with-amendment change set against its siblings. I found two LOW inconsistencies and no others.

**LOW 1. Amend text does not fit a Rename old-side finding.**
- Changed text: `cardNotDoneFor` now sets `createNotDone` for every `rename-not-done` finding, including the old side. The `recordbatch.go` `amend` string still says to amend the card's "Create target or Rename new side to the unit, owner or parameter list the finding names the member declared with".
- Evidence: `internal/planglyph/donecheck.go:434` words the old-side detail as "Rename pair's old side … still resolves — the rename did not happen". It names no unit or owner for the member to be declared with.
- For that finding the way forward points at the new side, but the old side is the part that failed. The review's `fix` accepted this, so it is only a wording mismatch.
- The `createNotDone` field comment now says "a Rename pair that did not happen", which is accurate.
- The first branch also tells the reader to run `recover-batch` "to build the target as the card names it", which is a looser fit for an old-side finding.

**LOW 2. Test comment not updated.**
- Changed text: the `TestFailCardNotDone_WayForward` doc comment in `recordbatch_test.go`.
- It still reads "It covers each Create-target arm: a signature mismatch, a plain missing target, a target after a recovery, an uncheckable record and an exhausted batch."
- The change added a Rename-pair arm and an unrendered-amendment arm, and the comment names neither.

**Checked and consistent:**
- The exhausted condition now matches `rendersNewAmendment` in `recoverbatch.go:386` and `:509`, `beginbatch.go:421` and `runlevel.go:1204`.
- The `doc.go` lines 331–332, the `failCardNotDone` doc comment, the `createTargetWayForward` doc comment and the `createNotDone` field comment all agree.
- `recoverbatch.go:732` is the only other caller of `failCardNotDone`, and it passes `Batch` and `Verb`, so it picks up both behaviours without a change.
- Nothing else in `internal/` or `contracts/` states the old create-only or Recoveries-only rule.

Tool calls:
1. `grep` for `failCardNotDone`, `create-not-done`, "exhausted-batch" and "used its counted recoveries" across `internal` and `contracts`. My context lacked the other callers and any other statement of the rule.
