# Card 9 — raddle-paths-and-keys

**Create:**
- `plan:internal/lymdirs#RaddleDirName` -> `const RaddleDirName = "raddle"`
- `plan:internal/loomengine#RaddleDirRel` -> `func RaddleDirRel() string`
- `plan:internal/loomengine#RaddleDir` -> `func RaddleDir(l *lymcwd.Location) string`
- `plan:internal/loomengine#RaddleProposal` -> `func RaddleProposal(l *lymcwd.Location) string`
- `plan:internal/loomengine#RaddleColdProposal` -> `func RaddleColdProposal(l *lymcwd.Location) string`
- `plan:internal/loomengine#RaddleReturnedProposal` -> `func RaddleReturnedProposal(l *lymcwd.Location) string`
- `plan:internal/loomengine#RaddleApproval` -> `func RaddleApproval(l *lymcwd.Location) string`
- `plan:internal/loomengine#RaddleStatePath` -> `func RaddleStatePath(l *lymcwd.Location) string`
- `plan:internal/loomengine#RaddleStateLockPath` -> `func RaddleStateLockPath(l *lymcwd.Location) string`
- `plan:internal/loomengine#RaddleStageDir` -> `func RaddleStageDir(l *lymcwd.Location) string`
- `plan:internal/loomengine#RaddlePartRecord` -> `func RaddlePartRecord(l *lymcwd.Location, slug string) string`

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine#TestPathsOutsideRunRecords`

**Extends:**
- `internal/lymdirs/doc.go`

**Uses:**
- `internal/loomengine#DiscussionDirRel`
- `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`

**Intent:** The raddle run's own files and its two loom keys.
`lymdirs.RaddleDirName` is the only place `raddle` is named under `_lym` (Constraint: `_lym/raddle/` named only in `internal/lymdirs`).
`RaddleDirRel` joins it onto `lymdirs.LymDirName`; `RaddleDir` anchors it at the anchor path, as `DiscussionDir` does, and the per-file accessors join the run's files onto it: `proposal.yaml` (the agent's proposal), `cold-proposal.yaml` (the cold proposal, stored once), `returned-proposal.yaml` (the proposal a return set aside for the next session), `approval.json` (the approval record `raddle-approve` writes), `state.json` (`RaddleStatePath`) (the run's raddle state) with its lock at the mirrored path under `lymdirs.DotLymDirName`, `stage/` (step 1's staged store) and `records/decision-<slug>.md` (step 2's part records).
`Config` gains `Raddle string` (`yaml:"raddle"`), the raddle agent's model-spec, and `RaddleAutoApprove bool` (`yaml:"raddle_auto_approve"`).
`template.yaml` gains `raddle: opus[high]  # model-spec for the raddle agent, which proposes how a Sketch splits into tasks (see contracts/specs/llm-model-spec.md)` beside `discussion`, set apart from it (D4), and `raddle_auto_approve: false  # true files a raddle proposal once its gate passes, with no operator approval; false, the default, has the agent put the proposal to the operator in its pane; read once, at Raddle-Write's first spawn in a run`.
`ConfigSpec`'s categories classify both keys `none`, as `discussion` and `discussion_interactive` are: `raddle_auto_approve` must stay open to the local layer, since D7 lets a run profile's raddle block set it and batten writes that block into the pair's local config; its bound is D7's own, the key is the sanctioned way to file with no approval and the mode is recorded once at Raddle-Write's first spawn, and the `ConfigSpec` doc comment names this exception to its guard rule.
`LoadConfig` parses `raddle` as a model-spec, as it parses `discussion`, refusing an invalid one naming the key.
`TestLoadConfig_Loads` asserts both defaults; `TestLoadConfig_Refuses` gains an invalid `raddle` row; `TestPathsOutsideRunRecords` gains the raddle accessors' rows.
`TestConfigTemplate_ContainsEveryConfigYAMLTag` and `TestModules_EveryTemplateKeyClassified` cover the two template lines and their categories as they stand.

**ImpactSummary:** Two keys added to loom's config and its key classification, and a set of path accessors; no reader exists until the raddle cards, and existing keys are untouched.
