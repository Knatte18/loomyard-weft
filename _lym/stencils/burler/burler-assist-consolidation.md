<!-- This is burler's assistant consolidation part: the review step's rules for a round with an assistant reviewer.
     The review step carries it after the cluster rules.
     `assistRulesBlock` (internal/burlerengine/assist.go) fills it.
     The result becomes the review step's optional assist_rules marker value.
     A round with no assistant renders the review step without it.
     It declares one required marker, {{.assist_origin}}, the origin value a kept assistant finding carries.
lym-stencil: sha256=8a6810e4c4e25bc880bec3855ce8538408f60b8a89b079822d422f896141a6f1 build=8cf7f477563f044e178a070cfd7dd27cb9f03c78 time=2026-11-23T09:03:29Z -->
### The assistant reviewer

An assistant reviewer reviews the same target beside you and writes one report; your prompt names its report and its end marker.
Before you consolidate, and before you write anything to the review file, wait for the assistant's end marker as your prompt says.
When the marker reads `done`, read the assistant's report and judge each of its findings with the same skepticism as a fork's: keep a finding that holds against what you read, tagged in its frontmatter with `origin: {{.assist_origin}}` (a finding that raises the plan executor's halt reason keeps `origin: webster` instead, and a `decision`-class finding that records an undriven or WARN focus entry keeps `origin: focus`), merge one that repeats a finding you keep, and move a false one to the `## Rejected` section with a one-line reason.
When the marker reads anything else, consolidate without the assistant.
The verdict is yours alone: the assistant's verdict is one more opinion, never a vote.
