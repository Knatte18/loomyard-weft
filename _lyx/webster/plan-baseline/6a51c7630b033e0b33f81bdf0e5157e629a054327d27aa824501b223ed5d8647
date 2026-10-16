# Card 8 — pattern-directive-changes

**Create:**
- `contracts/stencils/pattern/pattern-fragment-changes.md`
- `contracts/stencils#PatternFragmentChanges`
- `internal/pattern#changesMarker`
- `internal/pattern#changesFragmentStencil`
- `internal/pattern#renderChanges`
- `internal/pattern#fillDirective`

**Edit:**
- `internal/pattern#Directive` -> `func Directive(worktreeRoot, stencilsDir string, role Role, declared []string) (string, error)`
- `internal/pattern#FillDirective`
- `internal/pattern/doc.go`
- `internal/pattern#TestDirective_PerRole`
- `internal/pattern#TestDirective_RendersNothing`
- `internal/pattern#TestDirective_FilesystemFailuresAreErrors`
- `internal/pattern#TestDirective_EmptyRoot`
- `contracts/stencils/pattern/pattern-directive-implementer.md`
- `contracts/stencils/pattern/pattern-directive-review-fix.md`
- `contracts/stencils/pattern/pattern-directive-orchestrator.md`
- `contracts/stencils/pattern/pattern-directive-designer.md`
- `contracts/stencils/pattern/pattern-directive-judge.md`
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `.gitattributes`
- `internal/websterengine#RenderRecoveryPrompt`
- `internal/websterengine#RenderMasterPrompt`
- `internal/websterengine#MerriamBase`
- `internal/darnengine#DarnTable`
- `internal/burlerengine#Engine.Review`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/loomengine#DiscussionTable`
- `internal/loomengine#PlanTable`
- `internal/loomengine#ReworkTable`

**Intent:**
D5.4's render half: each PATTERN directive gains an optional `pattern_changes` marker, which `internal/pattern` fills with the task's declared changes and the rule that a listed entry is followed and judged by its listed new text, never the old.
`internal/pattern` stays a leaf (`PATTERN-leaf-packages`): the caller passes the declared changes in as rendered bullet lines, and this card's callers all pass nil, so every prompt renders byte-identically until cards 9 and 10 read the record.
Bound: a declaration lets every reviewer, judge and implementer follow and judge a listed entry by its declared text, never `PATTERN.md`'s current text;
it covers only the entries the decision record's `## PATTERN changes` lists, only through bullets of the shape `loom-discussion-spec.md` defines (card 5), appended after Discussion only through `lyx loom decision add`'s `--by parent|operator|arbiter` caller guards (card 7), and the Webster-Review rubric checks that the declared text lands (card 17).

- `pattern-fragment-changes.md` is a new fragment stencil with one required marker, `pattern_change_list`: a `### Declared PATTERN changes` heading, a line saying the task declares these changes to PATTERN entries and its own diff makes them, the marker, and the rule: an entry listed here is followed and judged by its listed new text, never by the overview's current text, an entry listed as `removes` no longer binds, and a departure from a listed entry cites the listed text.
  The rule also says that only the entries listed here are judged by their listed text, and every other entry by the overview's current text.
  `contracts/stencils` embeds it as `PatternFragmentChanges` with an `entries` row, `.gitattributes` gives it `text eol=lf` like the directive stencils, and `TestStencils_TopLevelMarkers` gains its row with `pattern_change_list`.
- Each of the five directive stencils gains `{{.pattern_changes}}` on its own line right after the `{{.pattern_overview}}` line, and its banner says the file declares two markers, the required `pattern_overview` and the optional `pattern_changes`.
- `Directive` gains `declared`: when PATTERN is active and the role known, `renderChanges` reads `changesFragmentStencil` through `stencilstore.Read`, strips its banner and fills `pattern_change_list` with the lines joined by newlines; nil or empty `declared` reads no fragment and renders the empty string.
  A fragment read or fill failure is returned wrapped like the directive stencil's own.
  An inactive PATTERN, an empty root or an unknown role still returns ("", nil) and reads nothing, whatever `declared` holds.
- `fillDirective` strips the banner and fills through `stencil.FillOptional` with `changesMarker` optional; `Directive` calls it with the rendered changes, and `FillDirective` keeps its signature and calls it with none, so `tools/tokencount`'s render of a historical stencil without the marker is unchanged.
- `doc.go`'s "one required marker" paragraph describes the second, optional marker, the fragment, and that the caller passes the declared changes in, read by `discussionparser.DeclaredPatternChanges`.
- Every caller listed passes nil for `declared`.

Tests:
`TestDirective_PerRole` keeps every role's checks with nil `declared`, its embedded-default comparison filling `pattern_changes` empty, and adds per role a call with one declared bullet whose directive holds the bullet and the fragment's heading after the overview.
Once, outside the role loop since `renderChanges` does not depend on the role, it calls `RoleImplementer` with that bullet over a stencils directory missing `pattern-fragment-changes`, which fails naming it, and over one whose `pattern-fragment-changes` carries an extra required marker, which fails naming the fragment's fill.
`TestDirective_RendersNothing` gains rows passing a declared bullet with an inactive PATTERN and with an unknown role, which still render nothing and read no stencil.
`TestDirective_EmptyRoot` gains a call passing a declared bullet, which returns ("", nil); `TestDirective_FilesystemFailuresAreErrors` passes nil.

**ImpactSummary:** a re-signed leaf function whose nine production callers pass nil here, so every rendered prompt is unchanged; the five directive stencils gain one optional marker.
