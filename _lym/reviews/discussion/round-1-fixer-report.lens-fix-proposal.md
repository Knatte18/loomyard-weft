```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I tested the reviewer proposals the fixer applied by building cases where they fail.

**MEDIUM — D4, first paragraph (F1, the fenced-body proposal).**
Changed text: "The body sits in one fenced `markdown` block whose backtick fence is one longer than the longest backtick run in the body, so no line of the body, a heading such as `## Decisions` or `## Open risks` included, starts a section: the discussion spec reads no fenced line".
The parser does not follow the CommonMark fence-length rule that this guarantee depends on.
- `fencedLines` in `internal/discussionparser/append.go:349-361` flips `inFence` on every line that starts with ```` ``` ```` or `~~~`, whatever its length.
- The section scanners skip only lines that `fencedLines` marks, for example `append.go:317-331` and `carryover.go:355`.

Two cases show the break:
1. **Inner code block.** A board body holds a fenced code block whose content has a `## Decisions` line.
   The outer longer fence opens. The inner opening ```` ``` ```` toggles back out of the fence, so that `## Decisions` line is read as a section heading. This is the exact failure F1 describes.
2. **Odd number of fence lines.** A body with an unclosed ```` ``` ````, or a stray `~~~` line, leaves an odd number of toggles. Every later line of the record is then marked fenced, including the record's own `## Scope`, `## Decisions` and the rest.
   `Validate`'s heading check then reports those sections missing. Record-Write writes a record its own gate refuses, and resuming never fixes it, because Record-Write never overwrites a record that exists.

Possible fixes:
- D4 names a change to `fencedLines` so it applies CommonMark's rule: a fence closes only on the same character, at least as long as the opening fence.
- Or Record-Write neutralizes body lines instead of fencing them, by demoting every heading so none becomes an H2 or H1. Note that demoting `#` to `##` would itself create an H2.

D4's Bound line and acceptance row would need to match whichever is chosen, and the acceptance row should include a body that holds its own code block and a body with an unclosed fence.

I found nothing to raise against the other proposals carried out as written (F2, F3, F7–F16).

Tool calls:
- Grepped `internal/discussionparser` for fence handling: my context held the discussion spec's claim that fenced lines are not read, but not the parser's fence rule.
- Grepped for the Preflight producer: to check F14's mapping to Preflight. My context had no view of the generic Preflight.
- Read `fencedLines` and listed the `## ` scanners: to confirm the toggle rule and which scanners use it.
- Read the `## ` scanners in `decisionids.go`, `append.go` and `identity.go`: to confirm they skip fenced lines, so the toggle rule alone decides.
- Read `internal/preflightshed/preflight.go`: to check whether Preflight refuses a run that already has history. It wraps `preflight.Check` over the working directory and reads no history, so F14's mapping holds.
