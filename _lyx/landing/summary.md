# Seats: the chair starts first and asks its advisors in rounds

The chair is the seats producer's main agent: it holds the gate, it is the only interactive seat, and its result decides the step.
`seatengine.Engine.Run` now starts it first and the advisors after it, in table order.

## Telling the chair who is up

An advisor may not exist yet when the chair is ready to ask its first batch.
Once each advisor's start ends, the engine queues one line on the chair's notice sender: `Advisor <strand> is up and takes your questions.`, or `Advisor <strand> did not start and never answers; decide without it.`
The sender now starts with the chair, so these lines go out through the same idle-gated, busy-retried path as the advisor death notices.
An advisor's watcher starts only after its up line is queued, so the chair never hears of an advisor's ending before it hears that the advisor is up.

These lines replace the `failed_advisors` reserved marker, which named failed advisors in the chair's prompt and cannot be filled when the chair starts first.
The marker is gone from `seatengine`'s reserved list, from `seatValues` and from `seat-directive-chair.md`.
`SeatResult.Notified` now also reports, for an advisor that never started, whether its not-started line landed.

`Engine.Resume` queues the up line again for every live advisor it attaches to, since the line may not have landed before the interruption; a repeated line costs the chair nothing.

## Failure paths

A chair that fails to start now returns before any advisor starts, so nothing has to be stopped.
An advisor whose misnamed strand cannot be stopped ends the run as before, and now stops the live chair along with every started advisor.

## Stencils

`seat-directive-chair.md` tells the chair it starts before its advisors, to send an advisor nothing before its up line arrives, and to end its turn if it is ready before then.
`loom-template-discussion-chair.md` states the round rule: a batch holds the questions that fit now, the chair waits for every advisor's answer, and it forms the next batch from those answers, never sending every question at once.
Claims in `contracts/stencils/claims_test.go` pin both rules.

## Tests

The engine tests now pin the chair-first start order, the up and not-started lines and their order relative to an advisor's ending, the chair stop on an advisor's unstoppable misnamed strand, and the re-sent up line on resume.
The fake shuttle's `holdStart` hook, used only by the removed "advisor ends before the chair exists" case, is deleted.
The seats smoke test, an `llm`-tier test, was not run.
