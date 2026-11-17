# Round 3 fixer report — discussion

## Changes

- F1 (MEDIUM, design), fixed differently in part.
  `_lym/discussion/decision-record.md` D4 gains a bullet after the Markdown-subset bullet:
  every other value the page renders from `window.BOARD` is inserted as text, never as markup,
  naming the header's render time and owner, project and entry fields, the facts line, section names, and design and decision file names.
  A design or decision file name reaches a link only as its `href` attribute set through the DOM, as a relative path opening with `./`.
  The D4 Review criterion gains "renders a title or brief holding an HTML tag as literal text"
  and "links design and decision files by a `./` relative path".
  Not made as proposed: the fix key's "with its double quotes escaped".
  The same bullet forbids building these values into markup, so the `href` is set through the DOM;
  on that path an escaped quote is stored literally and breaks the link, as the fix-proposal lens showed.
  The `./` prefix is added so a configured design-file prefix holding a colon cannot read as a URL scheme
  (`internal/boardengine/config.go:206-210`, per the fix-claims lens).

## Self-check

- fix-consistency, LOW: the new field list omitted status and project labels — fixed as proposed (list made non-exhaustive with "among them", status and project labels added).
- fix-consistency, LOW: the `href` rule had no acceptance criterion — fixed as proposed (D4 criterion names the `./` relative link).
- fix-proposal, LOW: quote escaping clashed with DOM-set `href` — fixed as proposed (the link is set through the DOM and the quote escaping dropped).
- fix-claims: no findings.
- fix-way-forward: no findings.

## Disputed

## Deferred

## Rulings
