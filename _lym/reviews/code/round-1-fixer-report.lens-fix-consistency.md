```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1–F7 change set against its surroundings.

**1. LOW: the thematic-break rule is stated more widely than the code applies it.**
- **Changed text:** the `continuesItem` doc ("unless it is an ATX heading, a thematic break…") and `thematicBreakLine` (`(-[ \t]*){3,}|(\*[ \t]*){3,}|…`). The same claim appears in the `SectionBullets` doc, the new sentence in `loom-discussion-spec.md` and `_lym/landing/summary.md` "## Change".
- **Problem:** the spaced forms `- - -` and `* * *` never reach `continuesItem`. Both `bulletLines` and `SectionBullets` test `isBulletLine` first, and it matches `- ` or `* `. Such a line therefore does not just end the bullet: it starts a new one, which `SectionBullets` returns as `- -` or `* *`. The raddle coverage check would then demand that a part carry it.
- **Evidence:** the `case isBulletLine(...)` arm comes before the `continuesItem` arm in both readers. The regex alternatives for spaced `-` and `*` are unreachable. `ThematicBreakEndsBullet` pins only `---`.
- **Context:** the bogus bullet predates this change. What the change added is documentation and a regex that read as if every thematic break were handled.

**2. LOW: the spec's lead sentence names only two terminators, then the next sentence adds more.**
- **Changed text:** `contracts/specs/loom-discussion-spec.md`: "A bullet runs on through the lines after it up to a blank line or the next bullet, and an indented line always continues it."
- **Problem:** the following sentence says a heading, thematic break, blockquote or HTML block also ends it. A fenced line ends it too, even an indented fence, which contradicts "an indented line always continues it".
- **Evidence:** `case fenced[i] || skipped[i]: flush()` in `partsplice.go` and `if fenced[i] { inItem = false }` in `bulletLines`. `summary.md` "## Change" does list "A blank line, the next bullet, a fenced block or a carry-over block still ends the bullet", so its twin in the spec omits two terminators.

I found no other twin of the bullet rule left stale. The `git grep` hits outside the changed files concern other formats (parts spec, partsparser, `PATTERN` projection, planparser).

Tool calls:
- One `git grep` over contracts, internal, docs and markdown for other statements of the continuation rule; my context held only the earlier symbol-name search, not a search for prose twins.
