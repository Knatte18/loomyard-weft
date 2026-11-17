```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the F1 change set agrees with the decision record around it (fix-consistency lens).

**[LOW] D4's new list of rendered fields leaves out status, which the row bullet says each row shows**
- Changed text: D4, the new bullet "Every other value the page renders from `window.BOARD` is inserted as text, never as markup: … each entry's handle, slug, title, brief and labels, the facts line, section names, and the design and decision file names."
- Evidence: the D4 bullet just above says "Each entry is a row with handle, title, status and labels". The colon-list reads as exhaustive, and it names every one of those fields except status. D1 also gives each project labels, and the list leaves those out too.
- The bullet's opening clause, "Every other value", still covers both fields. A reader who checks the list against the row bullet will still notice the gap.

**[LOW] The new `href` rule has no matching acceptance criterion**
- Changed text: D4, "A design or decision file name enters an `href` only as a relative path opening with `./`, with its double quotes escaped." This sits beside the D4 criterion addition "renders a title or brief holding an HTML tag as literal text".
- Evidence: the D4 Review criterion lists a check for the body's link rule ("keeps only `http`, `https` and scheme-less relative links, renders `javascript:` … as plain text"). The fix added a criterion for the text half of the new bullet but none for the file-name `href` half, so the bullet's second rule has no review check.

No further tool calls made.
