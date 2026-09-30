---
round: 1
focus:
  - "Comment Line-Break Convention (CONSTRAINTS.md): new and changed Go comments in internal/loomshed/planwrite.go (NewPlanDirRotator, rotateStalePlanDir), internal/shedadapters/singlellm.go (constructor doc, Call doc, the inline comment in Call) and internal/shedrecipe/entries_planwrite.go (planWriteEntry doc) - do they use semantic line breaks, or do they break mid-phrase at a fixed column? Judge only new or changed lines; untouched legacy wrapping is out of scope."
  - "Card 3 and card 6 Edit-group blast radius: does every non-nil supplier of the prepareFreshSpawn seam (production and test, including any closure still shaped func() error) compile and behave under func() (string, error), and does every construction path of the PlanWrite row (smoke harness, CLI wiring, integration fixtures beyond loomrecipe's buildSequenceFixture) set Env.StencilsDir and seed loom-template-prior-plan wherever a plan exists before Plan-Write runs?"
  - "Card 4 registry insertion in contracts/stencils/stencils.go entries: does anything besides internal/stencilcli's integration test index or count the registry by position or by a pinned total, so that inserting loom-template-prior-plan after loom-rubric-webster-review shifts an expectation for the burler and later rows?"
  - "internal/shuttleengine/claudeengine: does the --append-system-prompt argument survive sh.Quote and tmux send-keys on both the posix and pwsh shells (including the ':' and ';' in the notice sentences), and do command_test.go and prepare_test.go cover both the launch and the resume line for every deny combination as card 2 specifies?"
  - "Error wrapping on the prior-plan render path: loomengine.PriorPlanBlock wraps with 'loom: render prior-plan block' and NewPlanDirRotator wraps again with 'loomshed: render prior-plan block', and SingleLLMProducer.Call wraps once more - is the resulting message duplicated in a way the package's own error-prefix convention rules out?"
  - "Doc items named in card 6 and card 2: confirm internal/loomengine/plan.go's file comment names loomshed.NewPlanDirRotator as the rotation owner, internal/loomshed/doc.go describes the appended prior-plan block, and docs/overview.md's shuttle paragraph states the deny announcement, each consistent with what the code does."
---

The rubric's two added dimensions set the first two entries.
The repository's CONSTRAINTS.md declares a Comment Line-Break Convention for Go comments, and several changed doc comments in the diff visibly wrap mid-phrase, so comment compliance leads.
The per-card mechanical checks for this plan are Edit-group blast-radius checks, since the plan carries no Rename or Delete group;
cards 3 and 6 change a function type and add a required Env field, which is where an unchecked caller or fixture would break.
The remaining entries are ordinary diff-review questions on the registry order, the new command-line flag, error wrapping, and the docs the cards commit to.
