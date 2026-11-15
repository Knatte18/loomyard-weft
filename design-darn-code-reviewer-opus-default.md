# Darn's Code-Review reviewer should default to Opus, not Sonnet

`darn-code-reviewer-opus-default` · Task · enhancement, darn, profiles, loom · running · Describe

Imported from [issue #130](https://github.com/Knatte18/loomyard/issues/130).

## Seen

`code_reviewer_model` defaults to `sonnet[medium]` (internal/loomengine/template.yaml), and darn's Code-Review perch uses it.
In a darn run the Code-Review perch is the only in-run review: darn has no Discussion or Plan review, so this reviewer has to judge whether the writer did the task the entry asks for, not only whether the diff is correct.
In a recent darn run (a solver bug trace against a reference model, profile wide), the perch converged after three rounds and its last round rejected a finding that the entry's trace was left open as "not a defect".
The parent's PR review then found the work incomplete, which took three more fix rounds.
Hubs currently work around this per task with `run_settings: {"loom":{"code_reviewer_model":["fable[high]","sonnet[medium]"]}}`.

## Expected

Darn's Code-Review reviewer defaults to an Opus model-spec (for example `opus[high]`), set in darn's defaults or its profiles, so the in-run review can check the entry's work items, decisions and acceptance steps as well as the code.
Tapestry's Code-Review may keep its default, since its Discussion and Plan reviews already check the scope.