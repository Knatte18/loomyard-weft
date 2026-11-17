```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward the F1 fix introduced can be carried out and leads to progress.

No findings.

The fix added two things:
- **The new D4 bullet.** It is a rendering rule, not a refusal or a next step. Its `./`-prefixed relative `href` points to the same place as the "by relative path" link that D4's row bullet already states, so it opens no new path a reader can't follow.
- **The new D4 Review criterion** ("renders a title or brief holding an HTML tag as literal text"). A reviewer can check it by hand: give an entry a title holding a tag, render, and open `board.html`. The Constraints line, "the viewer's in-browser behaviour is a review check", already allows that kind of check.

Extra tool calls: none.
