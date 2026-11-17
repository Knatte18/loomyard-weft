```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the round's change set. Two docs were left as they were while the code they describe changed.

**MEDIUM — `internal/tenderengine/doc.go:97`**
- Text: "A slug with a live entry of its own scope gets none."
- Summary: the F3 change to `SyncBoltHold` makes this false for a `bolt-hold` entry that records another machine than `req.Machine`. That entry is now lifted and replaced.
- Evidence: the `SyncBoltHold` doc comment gained "A bolt-hold entry that records another machine than req.Machine is replaced…", but this package doc paragraph, which describes `SyncBoltHold`, was not updated. The comment on `SyncBoltHold` itself ("each slug of req.Hold with no live entry of its own scope gets a bolt-hold entry") also stays unqualified.

**LOW — `internal/dobbyengine/doc.go:541`**
- Text: "One notice names the hold or the standing one."
- Summary: F2 means an identical hold of this machine that already stands posts no notice, so the line is no longer unconditional.
- Evidence: `writeHold` in `upgrade.go` now posts only when `written.Wrote || written.Standing != nil`. The `writeHold` doc comment was updated; this twin was not.

**LOW — `internal/boardengine/doc.go:238-241`**
- Text: the `WriteHold` paragraph.
- Summary: F5 added "A hold of the same machine with another kind or other subjects is appended beside the standing one" to the `WriteHold` doc comment. The package doc paragraph does not state it.
- Evidence: the paragraph covers only the other-machine case and the identical-repeat case. The new contract line in `holdwrite.go` has no counterpart here.

No inconsistency found in the other sites I checked:
- The `OwnerCaller` comments at `record.go:100` and `record.go:242` are phrased neutrally about the entry's caller.
- The remaining `pauserecord/doc.go` mentions of `OwnerCaller` (line 31 and the gate paragraph) match the reworded legacy description.

Further tool calls: 4, 0 of them beyond the cap of 6.
- `grep` across `internal` for owner and hold wording, to find sibling mentions of the changed text that my context lacked.
- `grep` and `sed` on `boardengine/doc.go`, `pauserecord/record.go` and the tender and dobby docs, for the `WriteHold` and `OwnerCaller` siblings.
- `sed` on `tenderengine/doc.go:93-101`, since the package doc was not in my context.
- `sed` on `dobbyengine/doc.go:528-552` and `boardengine/doc.go:226-250`, to read the `UpgradeRead` and `WriteHold` doc paragraphs in full.
