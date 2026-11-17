```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal the fixer carried out in D4 and its criterion.

**Proposals checked:** F1, which adds the D4 bullet on rendering every other `window.BOARD` field as text, adds the `href` rule for file names, and adds the D4 criterion that a title or brief holding an HTML tag renders as literal text.

**LOW — D4's new bullet: "A design or decision file name enters an `href` only as a relative path opening with `./`, with its double quotes escaped."**
- Summary: the proposed quote escaping clashes with the sentence just before it.
- The case:
  - The bullet's first sentence says these values are "inserted as text, never as markup". An implementation that obeys it sets the link through the DOM, for example with `a.href = ...` or `setAttribute`, and builds no markup string.
  - On that path, escaping `"` to `&amp;quot;` is not neutral. The browser stores the literal characters `&amp;quot;`, so a file name holding a double quote produces a link to a file that does not exist.
  - Quote escaping only makes sense when the `href` is built inside a markup string. The body's Markdown path does that ("the escape covers double quotes, so an escaped target cannot leave its `href` attribute"), but the new bullet's own first sentence forbids it for these fields.
- Evidence: the two sentences of the new D4 bullet against each other, and against the existing body bullet's quote rule, which belongs to the string-built path.
- What would fix it: either say the link is set through the DOM's attribute API and drop the quote escaping, or say the quote escaping applies only when the `href` is written into markup.

I found no case against the criterion line: it is a review check that matches the bullet's text-insertion rule.

Further tool calls: none.
