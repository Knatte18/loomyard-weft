# Seats: the chair starts first, as the main agent of the producer

`seats-chair-first` · Note · enhancement, loom

## Rule (operator, 2026-10-11)

The chair is the seats producer's main agent: it runs the step, and whoever talks to a producer running seats talks to the chair.
The chair starts first, always; the advisors start after it.

## Seen

`seatengine.Engine.Run` starts the advisors in table order and the chair last (`internal/seatengine/doc.go`, "Running a table fresh"; `internal/seatengine/engine.go`).
The chair already holds the table's gate and is the only interactive seat, so the gate and the operator's channel are the chair's; only the start order is reversed.

## Idea

Start the chair first, then the advisors, and keep every rule that names the chair as the producer's face: its gate, its strand as the address for the run's messages, and its failure ending the step.
The chair may send its first batch before an advisor is up; the chair stencil or the engine has to cover that, for example by the engine telling the chair once each advisor is up, the way it already types advisor deaths into the chair.
