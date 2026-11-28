```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, checking that the fix covers every member of each finding's set.

**Left unchecked:** none. F4 needed no new search, since the parent's earlier grep already lists every `jacquard` assertion in `runprofile_test.go`.

**LOW** (changed text: `holdsAnchor` guard, `internal/runprofile/reinterpret.go:131`, F2)
- **Summary:** the anchor guard covers the jacquard block replacement only, not the two other sites where `ReinterpretJacquardBlocks` removes or replaces a node that may carry an anchor.
- **Sites:**
  - `reinterpret.go:183`: `setMappingValue(defaultsNode, ...)` replaces an existing `defaults.jacquard` scalar, anchor included.
  - `reinterpret.go:194`: `removeMappingKey(defaultsNode, ...)` deletes that scalar.
- **Case:** `defaults: {jacquard: &d docs, darn: *d}`, where `docs` holds only a darn block. The drop leaves `*d` with no anchor, so the rewritten file does not parse. This is F2's failure on the default instead of the block.
- **Evidence:** from reading the code; I did not run the case. It needs a hand-written anchor on a default, so it is unlikely.

F1 and F3: no gap in their sets.
- The existing "changed tapestry default" row already pins the `holdsJacquard[name] = held` bookkeeping.
- Outside this change: `ReinterpretArrasDefault` (`reinterpret.go:65`) has F3's half-drop shape, but the fix did not touch it.

**Tool calls:**
- One `grep` for the node write and removal helpers and `mergeFlag` across `internal`: my context had the file from before the edits, not a list of the sites after them.
