# Nested config keys, named in full with dots

All batches of the plan reached `done`, with HEAD at `fea915351951e0dd58b5903c19a733f681985d7f`.
By the plan's cards, the run built:

- `configengine.KeyName`, the moved-key refusal and `yamlengine.MoveKeys` (cards 1-3).
- Nested `loom.yaml` and `tapestry.yaml`, with readers, recipes and profiles on the nested paths (card 4).
- Move tables, and the moving of old keys in reconciled files, `profiles.yaml` and board `run_settings` (cards 5-8).
- A hub reconcile before loom and tapestry config loads outside `lym config` (card 9).
- Full key names in Go messages, template comments, contracts, `pattern/` and `docs/` (cards 10-14).

No digest listed a departure or a rename.
The scope guard listed these files as changed outside their batch's targets.
They are informational.

## 01-config-key-name

`cmd/lym/testonlycode_linux_test.go`; `internal/configengine/{config.go,layers.go,layers_test.go}`; `internal/yamlengine/{move.go,move_test.go}`.

## 04-nest-loom-tapestry

`contracts/stencils/stencils.go`; `internal/configcli/configcli_test.go`; `internal/configreg/{configreg_test.go,settinglookup.go,settinglookup_test.go}`; `internal/hardencli/cli_integration_test.go`; `internal/hubforge/hub_test.go`; `internal/landingshed/prreviewask.go`; `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml`; `internal/loomshed/{raddlewrite.go,raddlewrite_test.go}`; `internal/shedadapters/bouncer.go`.

## 05-moved-key-tables

`cmd/lym/testonlycode_linux_test.go`; `internal/configsync/configsync_test.go`; `internal/hubreconcile/ensure_integration_test.go`; `internal/loomengine/config_test.go`.

## 09-reconcile-before-load

`internal/loomcli/smoke_prebootstrap_integration_test.go`.

## 10-loom-key-messages

`internal/configcli/configcli_test.go`; `internal/configengine/layers_test.go`.

## 11-module-key-messages

- `internal/battenshed/{config.go,config_test.go}`, `internal/boardengine/{config.go,config_test.go}`
- `internal/burlercli/wiring_test.go`, `internal/configcli/configcli_test.go`
- `internal/configengine/{edit_test.go,layers_test.go,set_test.go}`
- `internal/desktophook/{config.go,config_test.go}`, `internal/envconfig/{config.go,config_test.go}`
- `internal/fabricengine/warpexclude.go`, `internal/heddleengine/shipped.go`
- `internal/ideconfig/{config.go,config_test.go}`, `internal/lenses/fankeys.go`
- `internal/loggerconfig/config.go`, `internal/loomengine/config_test.go`
- `internal/reedengine/{config.go,config_test.go,keynames.go,lifecycle_test.go,segmentcolor.go,serverlog.go,serverlog_test.go,tmuxconf.go,windowsize.go}`
- `internal/reportconfig/{config.go,config_test.go}`, `internal/samplerengine/{config.go,config_test.go}`
- `internal/seatengine/engine_test.go`, `internal/sheddriver/{config.go,config_test.go}`
- `internal/shellguard/rows.go`, `internal/shuttleengine/claudeengine/testdata/settings-golden.json`
- `internal/shuttleengines/{shuttleengines.go,shuttleengines_test.go}`
- `internal/slackengine/{config.go,config_test.go}`, `internal/weaverengine/config.go`

## 12-template-comments

None listed.

## 13-contracts-key-names

- `contracts/recipes/{darn,motif,sampler,tapestry}-recipe.yaml`
- `contracts/specs/{implementer-spec,landing-fix-review-spec,loom-final-summary-spec,loom-plan-checks-spec,loom-plan-spec,webster-spec}.md`
- `contracts/stencils/darn/{darn-fragment-make-change-gated,darn-fragment-make-change,darn-fragment-report,darn-template-write}.md`
- `contracts/stencils/drift/drift-template-judge.md`, `contracts/stencils/loom/{loom-block-plan-cards,loom-template-discussion}.md`
- `contracts/stencils/raddle/raddle-template-agent.md`, `contracts/stencils/webster/webster-body-implementer.md`
- `docs/dotnet-hub.md`, `docs/reference/claude-accounts.md`, `docs/shared-libs/lymcwd.md`
- `pattern/PATTERN-{agent-addressing,fabric-boundary,gate-slot,heddle-instructions,lym-files,reed-panes,shuttle-provider-seam,stencil-ownership,test-isolation,test-speed}.md`

## Verify gate findings

Evaluation 1 failed, with a verify step that hung until the test timeout.
The failing identities, under `internal/`:

- `configreg.TestPrepareRunSettings/hardener_concurrency_below_one_is_refused_naming_the_key`
- `orchcli.TestLoomPreBootstrapPair`
- `orchcli.TestOrchIntegration_Refusals/permission_mode/status` and `.../watch`
- the `loomcli` package, where the failure report named `TestLoomPreBootstrapPair`
- `shedverbs.TestLoopIntegration_ArmingRefusalStopsAsBootstrap/a_child_step_refuses_while_arming`
- `shedverbs.TestLoopIntegration_ArmingRefusalStopsAsBootstrap/the_loop's_keeper_refuses_while_arming`
- the `shedverbs` package
- `reedengine.TestAddStrand_ThePaneRunsInAScopeHeldToTheTemplatesMemoryCap`


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `57513a96976de817d01d74fadc9650f8e335716c` fix: stamp the arming fixture's hub, bound stuck loop steps and update the gate's failing assertions
