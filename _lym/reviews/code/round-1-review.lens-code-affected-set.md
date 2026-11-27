```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines they touch.

**Per change, the affected set:**
- There are no changes. HEAD is `3f83bb888aef38d1f9bd8d5d03c731661d6699d5`, which is the base itself. `git log` and `git diff --stat` over `3f83bb888..HEAD` both came back empty. With no changed signature, output, key, message, file shape or behaviour, the affected set is empty, so nothing is covered and nothing is left uncovered.

**Rubric checks under this lens:**
- **Stencil wording:** no stencil is changed. Nothing to check.
- **The repository's own rules:** no code, doc or rule-document change. No PATTERN entry is touched.
- **Declared PATTERN changes landed:** my directive declares no PATTERN changes. Nothing to check.
- **Commit messages and departures:** the range holds no commits. There is no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer to judge.

**Findings:** none.

**Tool calls:** none. The `git log` and `git diff --stat` runs in my context already settle that the range is empty.
