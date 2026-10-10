# Card 5 — discussion-table-builder

**Create:**
- `plan:internal/loomengine#DiscussionTable` -> `func DiscussionTable(layout *lyxcwd.Location, stencilsDir string, cfg Config, reg modelspec.Registry, slug string) (seatengine.Table, error)`
- `plan:internal/loomengine#discussionModeRules` -> `func discussionModeRules(autonomous, advisors bool) string`
- `plan:internal/loomengine#DiscussionAdvisorNotes` -> `func DiscussionAdvisorNotes(l *lyxcwd.Location, n int) string`
- `plan:internal/loomengine#discussionChairStencil` -> `const discussionChairStencil = "loom-template-discussion-chair"`
- `plan:internal/loomengine#discussionAdvisorStencil` -> `const discussionAdvisorStencil = "loom-template-discussion-advisor"`
- `plan:internal/loomengine#discussionAdvisorSkills` -> `var discussionAdvisorSkills []string`
- `plan:internal/loomengine#TestDiscussionTable` -> `func TestDiscussionTable(t *testing.T)`
- `plan:internal/loomengine#TestDiscussionTable_Refuses` -> `func TestDiscussionTable_Refuses(t *testing.T)`

**Edit:**
- `internal/loomengine#TestDiscussionPaths`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/loomengine#discussionRole`
- `internal/loomengine#discussionSkills`
- `internal/loomengine#modeRules`
- `internal/loomengine#DiscussionDecisionRecord`
- `internal/loomengine#DiscussionSupportLog`
- `internal/loomengine#DiscussionDir`
- `internal/loomengine#LoomFrictionDir`
- `internal/seatengine#Table`
- `plan:internal/seatengine#Prompts`

**Intent:** Build the seat-table producer's table in Go (D2), in a new file `internal/loomengine/discussiontable.go` with its tests in `discussiontable_test.go`, beside the untouched single-agent composer.
`DiscussionTable` refuses an empty slug, resolves `cfg.Discussion` through `reg` as `DiscussionSpec` does today, resolves each entry of `cfg.DiscussionAdvisors` the same way for the advisor seat's model, effort and version, returning `Resolve`'s error naming `discussion_advisors`, the 1-based index and the way forward "set the entry to a model-spec the registry defines" for a `Config` that did not come through `LoadConfig`, which already refuses such an entry at load, renders the designer pattern directive from the worktree root and the interview friction directive from the friction note path under `LoomFrictionDir` when `cfg.Friction` is set, swallowing a friction read failure to an empty string with the warning logged as today, and returns the table: role prefix `discussionRole`, segment discussion, timeout `discussion_timeout_min`, interactivity `cfg.DiscussionInteractive`, values `slug`, `decision_record_path`, `support_log_path`, `mode_rules`, `pattern_directive` and `friction_directive`, optional `pattern_directive` and `friction_directive`, and no gate, which the entry sets.
The chair seat names `discussionChairStencil`, the resolved discussion model, effort and version, outputs the decision record and the support log, inputs every advisor's notes file, and skills `discussionSkills`.
Advisor seat `n` names `discussionAdvisorStencil`, the `n`-th resolved advisor model, output `DiscussionAdvisorNotes(layout, n)` and skills `discussionAdvisorSkills`, which is `scribe:prose` alone.
The parent name is not a parameter: the seat engine renders every seat's parent directive from its geometry's parent name.
`DiscussionAdvisorNotes` returns `advisor-<n>.md` under `DiscussionDir`, loomengine staying the sole declarer of the path.
`discussionModeRules` returns `modeRules(autonomous)` and, in autonomous mode with advisors, appends one sentence: the chair's design questions go to the advisors named in its prompt, and what they cannot settle is self-picked; the two mode texts stay in `modeRules`, which the single-agent composer keeps.
`TestDiscussionTable` is table-driven over zero, one and two advisors and both interactivity values: it asserts the role prefix, segment, timeout, interactivity, the chair's outputs, inputs, skills and model, each advisor's output and skills, the optional list, that `Validate` passes against a stencilkit-seeded directory, and, through `seatengine.Prompts` with a geometry naming a parent, that the chair prompt names the parent, the board-read command, the slug and both paths, that the chair prompt names every advisor's strand name and notes path for one and two advisors and still composes with zero, that an advisor prompt names the chair as its parent in the non-interactive form and its notes path, that no prompt carries an unrendered marker, a skill name, `--auto` or an ask-the-operator sentence, that the mode text differs by interactivity, and that the autonomous chair prompt carries the advisor sentence exactly when the table has advisors.
Further rows: the table validates and every prompt composes with no `PATTERN.md` at the root, with `cfg.Friction` empty over a stencils directory without the directive stencils, and with an unreadable friction note directive; with `PATTERN.md` planted the chair prompt carries the designer directive before Step 1; and an on-disk edit of the chair stencil reaches the chair prompt.
`TestDiscussionTable_Refuses` covers an empty slug, a missing stencils directory naming the first stencil read, an unparsable chair model, and an advisor entry the registry does not define naming the key, the index and the way forward.
`TestDiscussionPaths` gains the advisor notes path for an anchored and an unanchored location.
The budget file's `internal/loomengine` row rises by two for the two new tests.

**ImpactSummary:** Additive: nothing calls the builder yet, the single-agent composer and its tests are untouched, and the one edited test gains rows only.
