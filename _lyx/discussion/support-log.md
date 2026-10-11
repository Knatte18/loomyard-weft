# Support log — twill-discusser

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick, listed in the Question ledger.

1. **Task and board.**
   Read `twill-discusser` and the entries around it: `parent-review-cold-cache` (the same wait verb, later), `discussion-multi-llm-bench` (the bench design and the plan to park `single` behind seats with no advisors), `discussion-advisor-mode` (lens advisors as chair forks), `fan-master-preread` (fork cost and the variant D fork text), `per-role-models` (a fork always runs on its parent's model), `plan-multi-llm` and `burler-multi-llm` (later consumers and the planned removal of `await-review`), `seats-chair-wait-notice`, `parent-review-hardening`.
   The bench report `docs/research/discussion-bench.md` supplied the cost picture.
2. **Engine boundary.**
   Read how `discussion_producer` reaches the recipe: `seats` swaps the row's engine; `single` runs `DiscussionSpec` through shuttle.
   Settled that `twill` is a `DiscussionSpec` variant (D1), since the forks live inside the writer's session.
3. **Prompt shape.**
   The discussion template carries the record format; a second full template would copy it.
   Settled on one optional marker in the existing template, filled by a reusable directive (D2, D3).
4. **Forks and shuttle.**
   Read the Claude settings denies (`ForkSubagents` installs a fork-only Agent allow and the webster fork guard) and shuttle's waiting-turn-end rule: a gated run's turn end with a fork outstanding is never an arrival.
   So the writer may end turns while its forks review, with no shuttle change (D4, D6).
5. **Wait mechanics.**
   Read `lyx burler await-review`: capped at 250 s, polls every 200 ms, relative paths through `lyxcwd.CwdFrom`.
   Settled a general `lyx shuttle await` sharing its poll, with `await-review` kept for fixers spawned before a deploy (D9).
6. **What the forks wait on.**
   Waiting on the decision record itself would wake a fork on a partially written or malformed file.
   Settled a ready marker the writer writes after the self-check passes, ephemeral under `.lyx`, never stale across spawns (D4, D7).
7. **Lenses.**
   Burler's `discussion` fan has `generic`, `goal-scope`, `pattern-fit`, `scenarios`, `leanness`; the last two cite the Discussion-Review rubric, which a twill fork would not otherwise have.
   Settled that the writer reads the rubric before spawning, a new fan `twill-discussion` with two new lenses (`premises`, `board-fit`) and no `generic` (D8).
8. **Cost bounds.**
   From `fan-master-preread`: fork cost follows fork turns and exploration; a fork told to verify against code runs up cost.
   Settled a 6-call budget with an accounting line per call, and a give-up after 15 capped waits (D5).
9. **Self-review against PATTERN.**
   Checked the decisions against the entries listed under Constraints; no entry is changed or waived.
10. **Scope addition from `ly:orch` (operator), after delivery.**
    The wait must be a general keep-cache-warm primitive, usable by a twill fork, a seats chair (the Burler fixer under `burler-multi-llm`) and a writer waiting on parent review, waiting on a file and ideally on a message or marker.
    Read the seats stencils: advisors answer the chair over SendMessage, which drains at the receiver's next tool round, so lyx cannot observe it.
    Reworked D9: the verb takes one or more paths and returns on the first that exists; a marker is a path; a message is not watched and arrives at most one cap late, and a sender needing an immediate wake also writes a marker.
    Answered `ly:orch` with that shape.

## Rejected alternatives

- **A new shed engine or seat-table producer for twill.** The forks are in-session subagents; shed sees one LLM, so a new engine adds wiring for nothing.
- **A separate full `loom-template-discussion-twill` stencil.** Copies the record format contract, against `PATTERN-producer-pointer-rule`, and drifts from `single`.
- **Forks wait on the decision record file itself.** Wakes on a partial write or a record that fails `validate-discussion`.
- **Spawning the forks right after exploration, before the design is settled.** More of an outsider view, but the forks would miss reads made while settling the design, and in interactive mode they would wait through the whole interview; the board entry says "ready to write".
- **Reusing Burler's `discussion` fan unchanged.** It has no lens for claims about existing code or for the rest of the board, the two checks a reviewer with the writer's exploration in context is best placed to make; `generic` overlaps the writer's own holistic view.
- **New self-contained copies of `scenarios` and `leanness`.** Would paraphrase the Discussion-Review rubric; reading the rubric once before spawning puts it in every fork's cached prefix and also sets the writer's own bar.
- **A loom key naming the twill fan, or a twill model key.** No requirement; `burler.yaml` already lets the operator redefine the fan, and forks always run on the parent's model.
- **Retiring `lyx burler await-review` in favour of the new verb.** A fixer spawned before a deploy would call a verb the new binary lacks; `burler-multi-llm` removes it with its wiring later.
- **The wait verb also watching for an incoming SendMessage.** lyx has no view of Claude Code's inbox, and a message queued during a foreground Bash call is delivered only when the call returns; the cap already bounds that delay, and a marker gives an immediate wake where one is needed.
- **One path per wait call.** A waiter with two exits, such as a reply or a stop marker, would need a verb per caller or a shell loop; a path list costs one argument shape.
- **The wait verb as a new `lyx` module, or under `loom`.** A new module costs sandbox coverage and docs for one verb; `loom` would not serve Burler or a later non-loom caller, while shuttle runs every agent.
- **A Go-side audit of fork writes, as Burler's cluster round does.** The record passes `validate-discussion`, parent review and Discussion-Review; the writer's own write scope is stencil-level too.
- **Re-forking after a gate re-prompt or a parent reject.** Doubles fork cost for a revision Discussion-Review will review anyway.
- **Unbounded fork waiting.** A writer that never writes the marker would keep six forks taking cached turns until the 480-minute deadline.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 (auto-pick).** New engine or spec variant? Spec variant of the single writer (D1): shed sees one LLM.
- **Q2 (auto-pick).** Separate template or optional marker? Optional marker filled by a reusable directive (D2, D3): no copy of the format contract.
- **Q3 (auto-pick).** When does the writer fork? After the design is settled, before writing either file (D4), per the board's "ready to write".
- **Q4 (auto-pick).** What do forks wait on? A ready marker written after the self-check passes (D4, D7).
- **Q5 (auto-pick).** How is a stale marker kept from waking new forks? Unique per spawn or removed before a fresh spawn; the Plan picks which (D7).
- **Q6 (auto-pick).** Which lenses? Fan `twill-discussion`: `premises`, `goal-scope`, `board-fit`, `pattern-fit`, `scenarios`, `leanness`, with the writer reading the review rubric first (D8).
- **Q7 (auto-pick).** Configurable lenses or model? Fan name fixed, editable through `burler.yaml`; forks on the `discussion` model; no new loom key (D8).
- **Q8 (auto-pick).** Where does the wait verb live, and what happens to `await-review`? `lyx shuttle await`; `await-review` kept over the shared poll (D9).
- **Q9 (auto-pick).** Fork cost bounds? Give up after 15 consecutive `ready: false`; at most 6 extra tool calls, each accounted for (D5).
- **Q10 (auto-pick).** Where are finding dispositions recorded? The last turn of the support log's `## Interview` (D4); `## Review rounds` stays the gate's.
- **Q11 (auto-pick).** Fork again after a re-prompt or reject? No: one fork pass per writer spawn (D4).
- **Q12 (auto-pick).** Audit fork writes in Go? No (D6); downstream gates bound the record.
- **Q13 (auto-pick).** Interactive mode? Same procedure; forks are spawned after the interview, so they do not wait through it.
- **Q15 (resolved by `ly:orch`, operator scope addition).** Is the wait Twill-specific? No: a general keep-cache-warm primitive for forks, seats chairs and writers on parent review (D9).
- **Q16 (auto-pick).** Which conditions does the general wait watch? One or more file paths, the first to appear ending the wait; messages are not watched, and a sender needing an immediate wake writes a marker (D9).
- **Q14 (open, for the bench).** Do six forks find enough to pay for themselves, and how much does inheriting the writer's reasoning blunt them? Measured by the post-landing bench.
