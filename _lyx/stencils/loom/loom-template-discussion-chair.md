<!-- This is the chair's opening prompt when the Discussion-Write row runs as a seat table (loom.yaml's discussion_producer: seats).
     internal/seatengine is its only filler, through stencil.FillWith, from the chair's value map.
     It declares no marker of its own; every marker reaches it through the two blocks it includes, each rendered whole from that same map:
     loom-template-discussion, the single writer's prompt, and seat-directive-chair.
lyx-stencil: sha256=e8c8bb79129fafb6fb066b87ce6469621482fb98b9026510020fd6bd6f97d3ee build=6df79b99d82d826539da6f216be524d984b39aea time=2026-10-11T08:24:01Z -->

{{template "loom-template-discussion"}}
{{template "seat-directive-chair"}}

## Working with your advisors

If your seat directive names advisors, you are not a single agent in this step: you use them as the rest of this section says, and that replaces the single-agent framing at the top of this prompt.
If it names none, you have no advisors: follow the steps above as a single agent, decide each question yourself as you judge best, and skip the rest of this section.
Every step, check and fence above still binds you alone.

- After Step 2, form your question batches exactly as Step 3 says, each question with your recommended answer and its alternatives, but keep the recommendation and the alternatives to yourself.
  Before you send a batch, write each question's private pick and the alternatives you saw into the question ledger, marked "private pick, before advisors".
  Send every advisor only the bare question: open and non-leading, naming no option, no recommendation and no hint of your lean.
  Give the context an advisor needs to research the question, never the answer space.
  No advisor owns an area, since each may find a different weakness.
- Weigh each advisor's answer against your private pick; an option an advisor names that you did not list counts as a new option.
  In the question ledger, record for each question whether the advisor's answer agreed with your private pick, differed from it, or added an option, and what you settled.
- Ask in rounds: a batch holds the questions that fit now, you wait for every advisor's answer to it, and you form the next batch from those answers.
  Never send every question at once.
- In autonomous mode, settle each question only after weighing every advisor's answer to it.
  An advisor that did not start, sent a failure notice or was written off counts as giving no answer, and a pick made with no usable answer stays an auto-pick.
- In interactive mode, put the batches where Step 4 says, and let the advisors supply facts.
  The operator's turns reach you alone.
- Under each turn of `## Interview`, carry every advisor answer you relied on, naming the advisor's strand name.
  In the question ledger, mark a pick that rests on such an answer as advisor-informed, naming the same strand name.
- Read each advisor's notes file once it exists.
  Write off an advisor that does not answer in reasonable time, and carry on after a failure notice.
- The step ends exactly as Step 6 says.
