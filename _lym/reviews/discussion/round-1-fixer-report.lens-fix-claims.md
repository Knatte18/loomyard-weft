```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: checking that every bound, behaviour or code fact the fix added or rewrote holds against the code at HEAD.

**BLOCKING: the D4 fence claim is false against the parser.** This leaves F1's premise unresolved for any body that holds a fence.
- **Changed text:** D4, "The body sits in one fenced `markdown` block whose backtick fence is one longer than the longest backtick run in the body, so no line of the body, a heading such as `## Decisions` or `## Open risks` included, starts a section". Its Bound line repeats it: "the board body sits fenced under `## Goal`".
- **Evidence:** `internal/discussionparser/append.go:349-361` `fencedLines` flips `inFence` on any line starting with three backticks or `~~~`. It ignores fence length and fence character.
- **Effect:** a body that holds a fenced code block of its own closes the outer fence at its first fence line. Every `## ` heading after the inner block's closing line is read as a section again. `h2Headings` (`carryover.go:349-355`), `ParsePatternChanges` (`patternchanges.go:68-86`) and the `## Decisions` insertion all use `fencedLines`.
- **Scope:** a board body with a code block is ordinary, so the D4 acceptance row about `## Decisions` and `## Open risks` holds only for bodies without inner fences.
- **Fix needs one of:** the parser's fence rule becomes CommonMark's (a closing fence at least as long as the opening, same character), and D4 says so. Or Record-Write neutralizes the body another way, for example by indenting or demoting every heading, and handles inner fence lines.

**MEDIUM: D13 credits `ValidateLocal` with a way forward it cannot give.**
- **Changed text:** D13, "`ValidateLocal` refuses one in a local layer, naming the key and the way forward, to set it under `shed` instead".
- **Evidence:** `internal/configengine/layers.go` gives a retired key whose category is not `CategoryNone` the generic message `key %q is %s and cannot be set in a local override`. It has no way forward.
- **Why it matters:** `RetiredKeys` maps a key to a `Category` only (`internal/configengine/doc.go:75`). The mechanism D13 names has nowhere to carry a per-key "set it under `shed`" way forward. D13's refusal would need a new spec field or a module-specific check, such as one in `PrepareRunSettings` or loom's own `ValidateLocalValues`. D13 does not say which.
- **Also affects:** the D13 acceptance row "naming the file, the key and the local `shed.yaml`" and the claim "a test reaches each" in the PATTERN-refusal-way-forward constraint.

These claims hold against the code:
- D3: `boardRecipe` in `internal/battencli/wire.go`.
- D10: the half-finished checks in `coherence.go`, `waivesHalfFinished`, and the `WriteSeed` disagreement refusal.
- D4: `CheckMainCapture`'s HEAD precondition, and `lym loom step` driving exactly one producer.
- D12: `CheckResolved` taking the run-wide budget, hard cap and amend values.
- D13: the refusal points `boardcli/runsettings.go:90` and `seamchild.go:150`, and profiles checked through configreg's `checkProfiles` CheckLoad.

Tool calls:
1. grep for fence handling in `internal/discussionparser`: the context did not show how fences are detected.
2. `fencedLines`, `missingSections` and the PATTERN parse's fence use: needed to test the D4 fence claim.
3. `ValidateLocal`'s retired-key message and the `PrepareRunSettings` callers: needed for D13's refusal claims.
4. Profile write paths and `lym loom step`'s help text: needed for D13's "every `profiles.yaml` write" and D4's `step` way forward.
5. `runprofile.go`'s functions: to find whether profile writes validate run settings.
6. `runprofile.ConfigSpec` and configreg's `runprofile` registration: to confirm `checkProfiles` CheckLoad covers profile writes.
