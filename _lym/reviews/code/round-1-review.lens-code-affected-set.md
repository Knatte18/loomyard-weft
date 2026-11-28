```yaml
lens: code-affected-set
outcome: reported
```

I worked the code-affected-set lens over the committed diff in `3fb0787e3..HEAD`.

**Per change, the set I found:** none. HEAD is `3fb0787e32473e6bd8d961da278a4bcecb4d2460`, the review base itself, so `git diff 3fb0787e3..HEAD` is empty: zero lines and no commits. The diff changes no signature, output, key, message, file shape or behaviour, so it forces nothing outside its lines.

The rubric checks this lens applies all came up empty:
- **Stencil wording:** the diff changes no stencil.
- **The repository's own rules:** the diff adds or changes no code or doc, so it breaks no PATTERN entry.
- **Declared PATTERN changes landed:** my directive declares no PATTERN changes, so I raise nothing.
- **Commit messages and departures:** the range holds no commits, so there is no trailer to judge.

**Findings:** none.

Tool calls:
- `git rev-parse HEAD` and `git diff &lt;base&gt;..HEAD | wc -l`: the earlier `--stat` output was empty but did not show the HEAD SHA, so I confirmed the range is empty.
