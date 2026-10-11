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

## Lens review by forks

Once your advisors' answers are weighed and the design is settled, run the procedure below; your advisors stay available while it runs.

When the design is settled and before you write any output file, run this procedure once:

1. Read the rubric at `/home/knatte/Code/loomyard-LYXHUB/_board/_lyx/stencils/loom/loom-rubric-discussion-review.md`, so every fork you spawn inherits it.
2. Spawn every lens below in one message through the Agent tool with `subagent_type: "fork"`, one fork per lens, and never pass a `name`.
   Each fork's prompt is the fork boilerplate below plus its lens text and nothing else: no design hint, no direction to read code.
3. Write the reviewed files per `Step 5`, and run `lyx loom validate-discussion` per `Step 6` until it passes.
4. Write the ready marker `/home/knatte/Code/loomyard-LYXHUB/{{.slug}}/.lyx/discussion/decision-record.md.ready` only after the self-check passes, creating its parent directory first.
   The forks wait for that file and review nothing before it exists.
5. Wait for every fork to return; ending your turn while forks are outstanding is allowed.
6. Judge every finding with the same skepticism, and merge duplicates across lenses.
   Fix the record, or reject the finding with a one-line reason.
7. Record the lens review at `the last turn of the support log's `## Interview` section`: one line per finding, with its lens and whether it was fixed or rejected and why.
   Name a fork that returned no report.
8. Run `lyx loom validate-discussion` again until it passes, then end your turn.

One fork pass per writer spawn.
A gate re-prompt, a parent-review reject or a later turn of the same session never spawns forks again.

A writer relaunched after its process died has lost every fork it spawned, and never spawns again.
On its next turn it resumes at the first of the steps from writing the files onward that it had not finished, with the findings it already holds.
It records at `the last turn of the support log's `## Interview` section` that the lens review was cut short, naming each fork that had not returned as "no report", or that no forks were spawned when it died before the spawn finished.

Lenses:
- premises: Focus on the record's premises: every claim it makes about existing code, config or behaviour matches what the code does, including one hop past what the record cites, such as the caller, the config key or the spawned process a cited piece leans on. Report anything else you notice too — emphasis, never exclusion.
- goal-scope: Focus on the record against its board entry: whether the stated goal is met, scope boundaries are explicit, nothing the entry asks for is missing, nothing unlicensed was added, no deferral or MVP phase hides in the text, and whether this is the right thing to build at all. Report anything else you notice too — emphasis, never exclusion.
- board-fit: Focus on the record against every other board entry: planned work it duplicates or contradicts, and a dependency it leaves unnamed. Report anything else you notice too — emphasis, never exclusion.
- pattern-fit: Focus on fit with the repo's rules: check every decision or card against the PATTERN.md entries and the pattern/ background files it touches, and flag a departure that does not name the entry it changes. Report anything else you notice too — emphasis, never exclusion.
- scenarios: Focus on scenarios: failure, restart and resume, concurrency, empty state and invalid input. Apply the Discussion-Review rubric's "The attack-surface question" and the discussion stencil's "Must-cover test scenarios go under `## Acceptance criteria`" rule. Report anything else you notice too — emphasis, never exclusion.
- leanness: Focus on content to move or cut: apply the Discussion-Review rubric's "Relocation and exclusion findings are legitimate, on equal footing with gap-filling findings" and "The completeness-before-leanness test". Seek what does not belong in the decision record rather than what is missing from it. Report anything else you notice too — emphasis, never exclusion.

### Fork boilerplate

Copy these lines verbatim into every fork's prompt, ahead of its lens text.

- Your first and only action until the file `/home/knatte/Code/loomyard-LYXHUB/{{.slug}}/.lyx/discussion/decision-record.md.ready` exists is `lyx shuttle await /home/knatte/Code/loomyard-LYXHUB/{{.slug}}/.lyx/discussion/decision-record.md.ready`, run with the Bash tool's `timeout` at 300000.
  On `ready: false`, run the same command again in a turn that does nothing else.
  Read nothing, judge nothing and write no note before `ready: true`.
- Give up after the wait has reported `ready: false` 15 times in a row, and return the one-line report "no record to review".
- After `ready: true`, read the files under `- `{{.decision_record_path}}`
- `{{.support_log_path}}`` and review them through your lens as an outsider.
- Your context already holds the writer's exploration and the rubric, so never re-read what your context shows.
  Make at most 6 further tool calls, each for something your lens needs that your context lacks.
  End your report with one line per such call, saying what you read and why your context lacked it.
- You are read-only: never write, edit or delete a file, run no git command, and never call the Agent tool.
- Return your findings only as your final message, each with a severity (`BLOCKING`, `MEDIUM` or `LOW`), the section or decision it concerns, a one-line summary and the evidence.

A fork never calls the Agent tool, so the writer is the only session that spawns.

The ready marker `/home/knatte/Code/loomyard-LYXHUB/{{.slug}}/.lyx/discussion/decision-record.md.ready` is a third file you create, and you create its parent directory too.
Where Step 5's lead-in names the two files as your write scope, and where its do-not-repair rule says "those two files", both include the marker.
The marker is untracked state that nobody commits, so the sentence leaving the commit of your two files to the loop owner stays about those two.
