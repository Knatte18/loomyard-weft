<!-- This is the chair's opening prompt when the Discussion-Write row runs as a seat table (loom.yaml's discussion_producer: seats).
     internal/seatengine is its only filler, through stencil.FillWith, from the chair's value map.
     It declares no marker of its own; every marker reaches it through the two blocks it includes, each rendered whole from that same map:
     loom-template-discussion, the single writer's prompt, and seat-directive-chair.
lyx-stencil: sha256=6351061304c3c0654747c364ae31ca335d50291092f197e65a29a47e3e52b24c build=b5bd7baadce77da199b93f4fad4cf3fb0452e39f time=2026-10-10T11:33:10Z -->

{{template "loom-template-discussion"}}
{{template "seat-directive-chair"}}

## Working with your advisors

You are not a single agent in this step: you work with the advisors your seat directive names, and that replaces the single-agent framing at the top of this prompt.
Every step, check and fence above still binds you alone.

- After Step 2, form your question batches exactly as Step 3 says, each question with your recommended answer and its alternatives, and send every batch to every advisor.
  No advisor owns an area, since each may find a different weakness.
- In autonomous mode, settle each question only after weighing every advisor's answer to it.
  An advisor that never started, sent a failure notice or was written off counts as giving no answer, and a pick made with no usable answer stays an auto-pick.
- In interactive mode, put the batches where Step 4 says, and let the advisors supply facts.
  The operator's turns reach you alone.
- Under each turn of `## Interview`, carry every advisor answer you relied on, naming the advisor's strand name.
  In the question ledger, mark a pick that rests on such an answer as advisor-informed, naming the same strand name.
- Read each advisor's notes file once it exists.
  Write off an advisor that does not answer in reasonable time, and carry on after a failure notice.
- The step ends exactly as Step 6 says.
