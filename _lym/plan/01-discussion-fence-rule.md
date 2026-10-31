# Card 1 — discussion-fence-rule

**Edit:**
- `internal/discussionparser#fencedLines`
- `internal/discussionparser#missingSections`
- `internal/discussionparser#TestAppendDecision_AppendsAtEndOfDecisionsSection`
- `internal/discussionparser#TestValidate_MissingHeadings`
- `contracts/specs/loom-discussion-spec.md`

**Uses:**
- `internal/discussionparser/doc.go`

**Intent:** Give the discussion parser CommonMark's fence rule for a fenced code block at the document's top level (D4), so a board body Record-Write later puts inside a fence can hold any heading or any code block without starting a record section.

`fencedLines` keeps its signature and its callers (append, carry-over, citations, identity, decision ids, PATTERN changes) and changes its rule:
- A fence opens on a line with at most three leading spaces followed by a run of at least three backticks or tildes; a backtick run whose remaining text (the info string) holds a backtick opens none.
- An open fence closes only on a line with at most three leading spaces, a run of the opening character at least as long as the opening run, and nothing after the run but spaces or tabs; any other line, a shorter run, the other character or a run followed by text such as ```` ```js ```` inside an open fence, is body text.
- A line indented four or more spaces neither opens nor closes.
- An unclosed fence runs to the end of the document.
- The parser reads no list item or blockquote, so a fence nested in one is read as a top-level line.

`missingSections` counts a required heading only on a line `fencedLines` reports outside a fence, so a fenced `## Open risks` never stands in for a record section that was removed; it keeps splitting the in-memory string (no scanner).

Test rows, as new cases of the two existing tests:
- `TestAppendDecision_AppendsAtEndOfDecisionsSection`: a record whose `## Goal` holds a four-backtick fence around a body with an inline-code line, a `## Decisions` line and a ```` ```js ````/```` ``` ```` inner block; one whose fence is indented three spaces around a `## Decisions` line; and one whose body opens an inner fence it never closes inside the four-backtick outer fence — each appends under the record's own `## Decisions`.
- `TestValidate_MissingHeadings`: a record whose only `## Open risks` line sits inside a fence reports that section missing; a record holding a ```` ```js ```` line inside an open fence followed by `## Decisions` and then the closing fence and the real sections reports nothing missing; an unclosed fence reports every later required section missing; a line indented four spaces that starts with three backticks opens no fence; a line ```` ```a`b ```` whose info string holds a backtick opens no fence, so a later `## Decisions` line counts; and a backtick fence followed by a `~~~` line and then `## Decisions` keeps that heading fenced, since a run of the other character never closes it.

The spec's "Lines inside fenced blocks are not read" sentence gains the rule above (opening, closing, indent, unclosed fence) as the definition of a fenced block, which the section check honours too.

**ImpactSummary:** Two parser helpers change rule, not signature; every record reader in the package inherits the stricter fence rule, and only fenced-line handling can differ.
