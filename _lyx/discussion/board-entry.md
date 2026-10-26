# Board entry `pattern-line-cap-text-only`: PATTERN line cap counts the rule text only, not the name and link

MaxEntryLineChars counts the whole entry line, so the room left for the rule text shrinks with the entry name and link

## Seen (umbrella-anchor Plan-Write, 2026-10-25)

`pattern.MaxEntryLineChars` (360) caps the whole overview line: the bullet, the entry name, the rule text, `(test)` and the background link.
The fixed parts took 79 characters for `PATTERN-cwd-resolution`, so its rule text had room for about 281, and a 331-character text recorded at Discussion failed the cap only at Plan-Write and needed a parent decision to shorten it.
The room for the text changes with the name length and with whether the entry has a link, which no writer sees when wording a rule.

## Idea

The cap counts the rule text alone, the part between the name and `(test)` or the link, so the limit a writer works against is one number whatever the name.
`lyx loom decision add --pattern-text` and the Discussion check measure the text against it when the decision is recorded, not at Plan-Write.
