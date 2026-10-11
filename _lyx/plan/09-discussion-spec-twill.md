# Card 9 — discussion-spec-twill

**Create:**
- `plan:internal/loomengine#twillFan` -> `const twillFan = "twill-discussion"`
- `plan:internal/loomengine#twillBlocks` -> `func twillBlocks(layout *lyxcwd.Location, stencilsDir string) (string, string, error)`
- `plan:internal/loomengine#TestDiscussionSpec_Twill` -> `func TestDiscussionSpec_Twill(t *testing.T)`

**Edit:**
- `internal/loomengine/discussion.go`
- `internal/loomengine#TestDiscussionSpec`
- `internal/loomengine#TestDiscussionSpec_Refuses`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `docs/overview.md`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/loomengine#DiscussionProducerTwill`
- `plan:internal/loomengine#DiscussionReadyMarker`
- `plan:internal/twilldirective#Directive`
- `plan:internal/twilldirective#WriteScope`
- `plan:internal/twilldirective#Values`
- `plan:internal/twilldirective#Lens`
- `internal/burlerengine#LoadConfig`
- `internal/burlerengine#ResolveFan`
- `internal/burlerengine#Lens`

**Intent:**
Make `twill` a spec variant of the single writer (decision record D1, D4, D6, D7, D8).

`twillBlocks`, in a new file `twill.go`, loads the worktree's burler config with `burlerengine.LoadConfig(layout.AnchorPath())` and resolves `twillFan` through `ResolveFan`.
A `burler.yaml` that cannot be read or parsed fails with an error naming the fan and `burler.yaml` whose way forward is to fix `burler.yaml`;
a fan that does not resolve, being undefined in both `burler.yaml` and the template or naming an undefined lens, fails with an error naming the fan and `burler.yaml` whose way forward is to define `twill-discussion` and its lenses there or restore the template's (`PATTERN-refusal-way-forward`).
It converts each `burlerengine.Lens` to a `twilldirective.Lens`, builds `twilldirective.Values` with the decision record and the support log as the reviewed files, `DiscussionReadyMarker(layout)`, the rubric path `stencilstore.Path(stencilsDir, "loom-rubric-discussion-review")`, `Step 5` as the format step, `Step 6` as the self-check step, `lyx loom validate-discussion` as the self-check command, and "the last turn of the support log's `## Interview` section" as the disposition place, then returns `twilldirective.Directive` and `twilldirective.WriteScope`.
Those literals are the Discussion host's own, so the directive stencil stays host-neutral.

In `discussion.go`, `DiscussionSpec`, when `cfg.DiscussionProducer == DiscussionProducerTwill`, calls `twillBlocks` and passes its two blocks to `composePrompt`, and sets `ForkSubagents: true`; every other field, the output files, model, effort, version, interactive flag, role, segment, skills and timeout, stays as for `single`.
For any other producer the two blocks are empty and `ForkSubagents` stays false.
`discussion.go`'s file header, the loomengine doc of the Discussion producer, records the twill design: with `twill` the spec is the single writer's with the twill block rendered into the discussion template and fork subagents authorized, the block's lenses come from the `twill-discussion` fan in `burler.yaml`, and the forks wait on the ready marker, which the writer creates once its record passes the self-check and Go removes before every fresh spawn.
The card targets `discussion.go` whole because it edits both `DiscussionSpec` and that header, which belongs to no member.

The twill edge and its bound: the spec sets `ForkSubagents`, so the writer may spawn subagents of type `fork`, and the `lyx webster` fork guard is installed whatever the Agent setting.
While the shuttle config's `claude_deny_agent_tool` is true, the template default, the Agent hook admits fork subagents only and refuses every other subagent type;
with it false no Agent hook is installed, so the writer and its forks may spawn any subagent type, as `single` may under the same setting.
The hook admits a fork spawn from any session in the run, so a fork spawning a further fork is refused only by the fork boilerplate, not by the hook.
Forks inherit every other standing deny of the writer's session.
The writer's write scope and a fork's read-only and no-Agent-call rules are stencil-level, and the discussion gates bound what lands in the record, reading the two discussion files only.

`refusal-spec.md`'s `## loom` section gains two rows beside `config: unknown fan` and `config: burler.yaml unreadable`, each a correctness halt quoting its message's way forward: `Discussion-Write: burler.yaml unreadable for twill`, triggered when `discussion_producer` is `twill` and `burler.yaml` cannot be read or parsed at the writer's spawn, whose way forward is to fix `burler.yaml`;
and `Discussion-Write: twill-discussion fan unresolvable`, triggered when `discussion_producer` is `twill` and `twill-discussion` is defined in neither `burler.yaml` nor the template or names an undefined lens, whose way forward is to define `twill-discussion` and its lenses in `burler.yaml` or restore the template's.

`TestDiscussionSpec` gains a producer column: its `single` rows assert `ForkSubagents` is false, and a `twill` row asserts the same output files, interactive flag, role, segment, model, effort and timeout with `ForkSubagents` true.
`TestDiscussionSpec_Refuses` gains a hub-shaped temp-dir location whose anchor holds `_lyx/config/`, and two `twill` rows: a `burler.yaml` that does not parse, and one redefining `twill-discussion` to name an undefined lens, each failing with an error naming `twill-discussion` and `burler.yaml`.
`TestDiscussionSpec_Twill` is a new untagged test, covering only the rendered twill block and the operator override, with its own seeded stencils directory and the same kind of temp-dir location:
with `twill` and no `burler.yaml`, the prompt carries every lens name of the template's `twill-discussion` fan in order, `DiscussionReadyMarker(layout)`, the rubric path, `lyx shuttle await`, `subagent_type: "fork"`, `lyx loom validate-discussion`, `Step 5`, `Step 6` and the write-scope sentence about the marker's parent directory, with the directive between the mode rules and `## Step 5`;
and a `burler.yaml` at the anchor redefining `twill-discussion` to `[goal-scope]` renders that lens alone.

`test-budget.yaml` raises `internal/loomengine` by one for the new test.

`docs/overview.md`: the loom config line says `discussion_producer` is `single`, `seats` or `twill`; the Discussion producer paragraph gains the third producer, the writer that settles its design, forks itself once per lens of the `twill-discussion` fan, writes, marks the record ready, takes the forks' findings in and finalises, with the forks waiting on `lyx shuttle await` so their prompt cache stays warm.

**ImpactSummary:** `DiscussionSpec` has one production caller, the `loomcli` wiring closure, whose call does not change; `single` and `seats` render as before.
