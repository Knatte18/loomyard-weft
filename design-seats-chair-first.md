# Seats: the chair starts first and asks its advisors in rounds

`seats-chair-first` · Task · enhancement, loom · awaiting · PR-Gate

## Rule (operator, 2026-10-11)

The chair is the seats producer's main agent: it runs the step, and whoever talks to a producer running seats talks to the chair.
The chair starts first, always; the advisors start after it.

The chair asks in rounds: it sends a batch of the questions that fit now, waits for the answers, and forms the next batch from them; it never sends every question at once.

## Seen

`seatengine.Engine.Run` starts the advisors in table order and the chair last (`internal/seatengine/doc.go`, "Running a table fresh"; `internal/seatengine/engine.go`).
The chair already holds the table's gate and is the only interactive seat, so the gate and the operator's channel are the chair's; only the start order is reversed.

The chair stencil (`contracts/stencils/loom/loom-template-discussion-chair.md`) says to form question batches "exactly as Step 3 says" and send every batch to every advisor, and to settle each question only after weighing every answer.
Step 3 of `loom-template-discussion.md` asks for "focused batches", which is right, but neither stencil says to wait for a batch's answers before forming the next from them, so nothing stops a chair sending all its questions in one go.

## Idea

Start the chair first, then the advisors, and keep every rule that names the chair as the producer's face: its gate, its strand as the address for the run's messages, and its failure ending the step.
The chair may send its first batch before an advisor is up; the chair stencil or the engine has to cover that, for example by the engine telling the chair once each advisor is up, the way it already types advisor deaths into the chair.
The chair stencil states the round rule: a batch holds the questions that fit now, the chair waits for every advisor's answer to it, and the next batch builds on those answers.
