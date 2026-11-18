```yaml
lens: code-affected-set
outcome: reported
```

I reviewed the code-affected-set lens: what the diff's changes force outside the lines it touches.

**Per change, the affected set and whether the diff covers it**

1. **Claude edit directive** (`contracts/stencils/engine/claude/edit-directive.md:9`): the `cat`, `head` and `sed -n` override clause is gone.
   - `internal/editdirective/doc.go:13` is the package's account of the override. The diff updates it to "changing files by shell", so it is covered.
   - `contracts/stencils/engine/codex/edit-directive.md:9,13` is the Codex copy. Its override already covered only changing files, and line 13 already allows reading with `cat`. It is consistent and needed no change.
   - `pattern/PATTERN-stencil-ownership.md:78-86`, "Edit directive", says each engine copy carries every rule in its own tool wording. Both copies now carry the same override rule, so it is covered.
   - `pattern/PATTERN-engine-parity.md:12,24` reads the edit directive off the rendered prompt as a channel, not by its wording. The diff does not affect it.
   - `internal/shuttleparity/compare_test.go:110-119` builds its expectations from the rendered directive through `editdirective.Directive`. It does not assert the wording, so it is unaffected.
   - The `internal/editdirective` tests and Go code have no reference to "harness", "reading" or `cat`, so nothing there depends on the removed wording.
   - `internal/shuttleengine/claudeengine/doc.go:355` counts `cat`, `head` and `sed -n` as read-only lookups for the batching nudge. That is consistent with the change: reading by shell is no longer forbidden.
   - `internal/shellguard/rows.go:75`, the Codex rule string that allows `cat` and `sed -n` reads, is consistent.
   - No other stencil, spec, PATTERN entry or heddle text carries the removed clause; a repo-wide grep for "overrides any harness" finds only the two engine copies and `doc.go`.
   - `contracts/stencils/edit/edit-directive.md`, which the Goal names, does not exist. Nothing to change there.

2. **New rubric bullet "An unpinned tool fact"** (`contracts/stencils/loom/loom-rubric-discussion-review.md:43-46`).
   - `internal/loomengine/discussiontable_test.go:547` asserts only the rubric's path, never its bullets, so it is unaffected.
   - The writer's stencil, `contracts/stencils/loom/loom-template-discussion.md:78-82`, tells the writer to record a researched fact "with the evidence it rests on, named by source and version". The new bullet accepts the tool's documentation as a pin, which matches that. The writer is never told not to gather this, so the rubric's writer/reviewer symmetry note is not broken. Covered.
   - No lens file or spec restates the rubric's research bullets; grep finds them only in the rubric itself.

3. **Repository rules, declared PATTERN changes and commit messages**
   - The directive declares no PATTERN changes, and the diff touches neither `PATTERN.md` nor `pattern/`.
   - Neither commit carries a `Pattern-Departure:` trailer, and none is needed.
   - Both commit messages describe their diff accurately.
   - The edited stencil keeps its marker-free header comment, as `PATTERN-stencil-ownership` asks, and `doc.go` still documents only its own package, as `PATTERN-code-conventions` asks.

**Findings**

None.

**Tool calls**

1. A repo-wide grep for "overrides any harness", "harness text" and the `cat`/`head` clause, to find docs outside the manifest that mirror the removed clause.
2. A grep of `PATTERN.md`, `pattern/` and `docs/specs` for the edit directive, plus a search for the rubric's research bullets, to find rule documents and tests describing either change.
3. A read of `pattern/PATTERN-stencil-ownership.md:76-100` and a search for tests naming the rubric, because the manifest lacked the directive's PATTERN background and any rubric test.
4. A read of `internal/loomengine/discussiontable_test.go:535-560`, to see whether that test asserts the rubric's bullet text.
5. A grep of `contracts` for spec or lens text restating the research or tool-behavior bullets, which the manifest did not cover.
