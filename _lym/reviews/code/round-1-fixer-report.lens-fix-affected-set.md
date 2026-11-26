```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the fix, rerunning the searches behind the set-shaped findings.

**Findings left unchecked:** none. F1, F3, F5 and F4 with F7 were searched (the last two shared a search). F2 and F6 are single-test coverage gaps with no set to search.

1. **MEDIUM: moved-check rule (`drift-template-judge.md` Step 2, `doc.go:239`, fix F1).** Code-Review still treats a moved check with no trailer as a finding.
   - `contracts/stencils/loom/loom-rubric-code-review.md:85` says "An assertion, tolerance or limit the diff widens or removes in a test that exists at the review base is a finding, unless a `Loosened-Check:` trailer … names it". It has no exception for a check another test holds.
   - `contracts/stencils/loom/loom-fragment-loosened-check.md:7` likewise asks for a trailer on every removal.
   - So the halt's third way forward (commit the dropped checks in a test, with no trailer) now passes Drift, but the following Code-Review, as its rubric is worded, raises the same removal. The two judges of one rule disagree.

2. **LOW: re-judge trigger wording (`_lym/landing/summary.md`, fix F5).** Two sites still claim that any new commit re-judges.
   - The title reads "re-judges an uncaused loosened ruling after a new commit".
   - The second Design decisions bullet reads "any new commit is the writer's attempt to answer it, so one judge spawn per new commit is the right cost". The key hashes only file paths and diffs, so a commit that changes neither spawns no judge for it.

3. **LOW: `internal/driftengine/doc.go:243`, same set as F5 and F7.** "reused only while the whole range is unchanged" can be read to include commit messages. After F7 the range key covers only file paths and diffs; a trailer change re-lists files through each file's own key.

The other sites hold:
- `docs/overview.md:1372` inherits the corrected `:1122` through "the same way".
- `rangeRulingKey` has one caller, and it is updated.
- The comments at `loosened.go:82`, `records.go:176` and `verify.go:257` agree with the code.

Tool calls:
- `git grep` for loosened/uncaused lines across docs, contracts and internal: my context held only the parent's narrower grep, without the F1 sibling terms.
- `git grep` for reuse and new-commit wording and `RangeKey` in `driftengine`, docs and contracts: my context lacked a search of these phrasings.
