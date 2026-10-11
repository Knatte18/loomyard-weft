# Card 8 — discussion-spec-twill

**Create:**
- `plan:internal/loomengine#twillFan` -> `const twillFan = "twill-discussion"`
- `plan:internal/loomengine#DiscussionReadyMarker` -> `func DiscussionReadyMarker(l *lyxcwd.Location) string`
- `plan:internal/loomengine#twillBlocks` -> `func twillBlocks(layout *lyxcwd.Location, stencilsDir string) (string, string, error)`
- `plan:internal/loomengine#TestDiscussionSpec_Twill` -> `func TestDiscussionSpec_Twill(t *testing.T)`

**Edit:**
- `internal/loomengine#DiscussionSpec`
- `internal/loomengine#TestDiscussionSpec`
- `internal/loomengine#TestDiscussionPaths`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `plan:internal/loomengine#DiscussionProducerTwill`
- `plan:internal/twilldirective#Directive`
- `plan:internal/twilldirective#WriteScope`
- `plan:internal/twilldirective#Values`
- `plan:internal/twilldirective#Lens`
- `internal/burlerengine#LoadConfig`
- `internal/burlerengine#ResolveFan`
- `internal/burlerengine#Lens`
- `internal/loomengine#discussionDirName`

**Intent:**
Make `twill` a spec variant of the single writer (decision record D1, D4, D6, D7, D8).

`DiscussionReadyMarker`, in `config.go` beside the other discussion path accessors, returns the marker path under the anchor's `lyxdirs.DotLyxDirName` at the subpath mirroring the discussion directory, `discussionDirName` then `decision-record.md.ready`, spelling neither lyx directory literal itself (`PATTERN-lyxdirs-single-declarer`, `PATTERN-durable-vs-ephemeral-state`).

`twillBlocks`, in a new file `twill.go`, loads the worktree's burler config with `burlerengine.LoadConfig(layout.AnchorPath())`, resolves `twillFan` through `ResolveFan`, and on either failure returns an error naming the fan and `burler.yaml` as the way forward: define the fan and its lenses there or restore the template's.
It converts each `burlerengine.Lens` to a `twilldirective.Lens`, builds `twilldirective.Values` with the decision record and the support log as the reviewed files, `DiscussionReadyMarker(layout)`, the rubric path `stencilstore.Path(stencilsDir, "loom-rubric-discussion-review")`, `Step 5` as the format step, `Step 6` as the self-check step, `lyx loom validate-discussion` as the self-check command, and "the last turn of the support log's `## Interview` section" as the disposition place, then returns `twilldirective.Directive` and `twilldirective.WriteScope`.
Those literals are the Discussion host's own, so the directive stencil stays host-neutral.

`DiscussionSpec`, when `cfg.DiscussionProducer == DiscussionProducerTwill`, calls `twillBlocks` and passes its two blocks to `composePrompt`, and sets `ForkSubagents: true`; every other field, the output files, model, effort, version, interactive flag, role, segment, skills and timeout, stays as for `single`.
For any other producer the two blocks are empty and `ForkSubagents` stays false.
The file header says the spec renders the twill block for `twill`.
The twill edge's bound, for the record: the spec authorizes fork subagents only, which the engine admits while refusing every other subagent type under the config's Agent deny; the writer's write scope and a fork's read-only rule are stencil-level, and the discussion gates bound what lands in the record.

`TestDiscussionSpec_Twill` is a new untagged test with its own seeded stencils directory and a hub-shaped location under a temp dir whose anchor holds `_lyx/config/`: with `twill`, the spec sets `ForkSubagents`, keeps `single`'s output files, role, segment, skills, model, effort and timeout, and its prompt carries every lens name of the template's `twill-discussion` fan in order, `DiscussionReadyMarker(layout)`, the rubric path, `lyx shuttle await`, `subagent_type: "fork"`, `lyx loom validate-discussion`, `Step 5`, `Step 6` and the write-scope sentence about the marker's parent directory, with the directive between the mode rules and `## Step 5`;
with `single`, the prompt carries no `twill` text and `ForkSubagents` is false;
a `burler.yaml` at the anchor redefining `twill-discussion` to `[goal-scope]` renders that lens alone;
a `burler.yaml` redefining it to name an undefined lens fails the spec with an error naming `twill-discussion` and `burler.yaml`.
`TestDiscussionSpec` asserts `ForkSubagents` is false in both of its rows.
`TestDiscussionPaths` asserts `DiscussionReadyMarker` sits under the anchor's `.lyx` at `discussion/decision-record.md.ready`, spelled through `lyxdirs.DotLyxDirName`.

`test-budget.yaml` raises `internal/loomengine` by one for the new test.

`docs/overview.md`: the loom config line says `discussion_producer` is `single`, `seats` or `twill`; the Discussion producer paragraph gains the third producer, the writer that settles its design, forks itself once per lens of the `twill-discussion` fan, writes, marks the record ready, takes the forks' findings in and finalises, with the forks waiting on `lyx shuttle await` so their prompt cache stays warm.

**ImpactSummary:** `DiscussionSpec` has one production caller, the `loomcli` wiring closure, whose call does not change; `single` and `seats` render as before.
