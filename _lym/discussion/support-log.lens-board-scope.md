```yaml
lens: board-scope
outcome: reported
```

I reviewed the record's fit with the board: the entry's goal points, the touching entries, peer runs, parent commits and decision notes. I found no BLOCKING finding.

**Goal points of the entry**
- Records on their own branches in the code repository: met (D1, branches under `refs/lym/heads/`).
- Wired as `_lym` like weft: met (D1, the records store is the weft sibling with junctions).
- The layout known only to fabric: met (D2).
- Decide what of f06acf39 stays: met (D5, D6, D7).
- Weigh a custom ref namespace against plain branches, per the operator's scope note: met. D1 picks the namespace, and the support log rejects prefixed branches.
- The two-repository hub stays the standard: met (Scope Out, D1).

**Entries, peers and commits checked**
- `two-repo-merge-probe-regression` (running darn): conflict over files; findings 1 and 2.
- `single-repo-hub` (project P4, parent of this task): stale premise; finding 3.
- `drop-compat-migration-code`: none. D10 follows its ruling against migration code.
- `board-viewer-build`: none.
- `empty-landing-not-flagged`: overlaps only in text, on `internal/fabricengine/parenttips.go` and `internal/landingshed/finalize.go`. No finding.
- `parent-review-code-and-drift`: overlaps only in text, on `internal/loomcli/wiring.go`. No finding.
- `loop-ends-without-envelope`: none.
- Parent commits: `lym loom peers` lists none.
- Decision notes: no `Decision for single-repo-records-branches:` note appears in `lym board list`. None was unreachable.

**Findings**

1. MEDIUM, class `decision` (Open risks, D2, D5): **conflict with a running peer.**
   - The peer darn `two-repo-merge-probe-regression` adds a cache and a quiet read to `oneRepositoryAt` in `internal/fabricengine/onerepository.go`.
   - It also edits `mergestateactive.go` and `recordsduringmerge.go`.
   - D5 deletes those files, or the code in them, and D2 replaces the probe.
   - Question for the parent: should this run wait for the darn to land, or should the darn be narrowed to the merge-probe revert alone? The probe cache it builds is thrown away here.
   - The record already raises this under Open risks; this finding routes it to the parent.

2. LOW, class `scope` (D7): **D7 repeats work the peer owns and pins a different case.**
   - The peer's goal makes the merge-state probe report a merge only, and its test is a status commit during a weft cherry-pick.
   - D7 redoes that revert, and its criterion pins a cherry-pick "in the code worktree".
   - Fix: say D7 takes the peer's revert as landed, and cover only the one-repository case it adds.

3. LOW, class `scope` (record overall): **the project's title is outdated.**
   - The parent project `single-repo-hub` is titled "records on each code branch, the board on lym-bolt".
   - Its Sketch's D8 and D9 (the records-free pull request view and the records-stripping code identity) are what D1 makes needless.
   - The record changes the project's premise but says nothing about its stale title, or about any remaining entry such as `single-repo-code-identity` should one exist.
   - Fix: a decision to retitle the project entry, or a note for the parent. Changing another entry is within the rules.

**Tool calls beyond my context**
- I read the peer's `decision-record.md` in `two-repo-merge-probe-regression-weft/_lym/discussion` to compare its goal against D7; my context held only its board body.
