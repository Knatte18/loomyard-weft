---
round: 1
focus:
  - "internal/boardengine/handles.go and its call sites (Store.Load, boardCriticalSection, FabricReplay.Apply): does assignHandles keep a handle fixed while its entry lives, reuse freed numbers, and reassign on promote, as the entry's decision 2 says?"
  - "Existing entries get handles once, in creation order: check that load-time assignment in id order matches this and that the first write persists exactly what reads showed."
  - "Every verb that took a slug or id also takes a handle (decision 3): compare the verbs in internal/boardcli, including merge set_status, against the `selects` matcher users; look for a verb or selector left slug/id-only."
  - "Tasks carry no priority (decision 1): check normalizeTask refusal, Promote clearing the note's priority, and the load-time drop in migrateEntries against the run lock and revised date."
  - "README rendering against decision 4: handle replaces row number, one table per note type sorted by handle, `P` column H/L/blank, merged `Age<br>Revised` cell with two-line header, tasks keep running/ready/layer groups sorted by handle."
  - "`lyx board list --text` prints the handle first (decision 5): check the case where the column is omitted when no listed entry has a handle."
  - "Repository rules: doc.go updates for boardengine and boardcli, PATTERN.md if a cross-cutting invariant is new, and that the diff does not change the task's own board entry body or scope."
  - "Summary claims in _lyx/landing/summary.md (LoadConfig refusals, bare-handle argument, upsert refusal of `handle`, replay dropping a local handle) against the diff."
  - "Test economy: new handles_test.go cases against existing tests that already cover the same behavior, and any added behavior left without a test."
---

Seeded from the board entry and the change description; no prior round exists.
