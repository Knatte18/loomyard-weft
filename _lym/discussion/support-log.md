# Support log — hold-notice-operator-turn

## Interview

A darn run holds no interview; the board entry is the whole statement of the task.

## Rejected alternatives

- Name, in the notice, who the agent talked to, and keep sending it: the entry asks for no notice for an operator turn, and the parent has nothing to act on there.
- Track the sender of each send in memory in the waiting process: creel mail is sent by another process, and an attach would lose the memory; the sent-texts file already holds every send.
- Read the pane or the transcript for the operator's input: provider-specific, and the hook's turn start already carries the prompt text.
- A new per-send marker in the typed text: changes what agents and the operator see, and the sent-texts record already answers the question.

## Review rounds

_No rounds yet._

## Question ledger

- Should the notice for an operator turn be deferred instead of dropped? Self-picked: dropped; a later held end of a turn lym starts is notified, so a real stall still reaches the parent.
- Should a turn start with no instruction (a task notification or slash command) count as the operator's? Self-picked: no; it is no typed instruction, and a task notification's turn is the agent's own work.
- Should the run's prompt turn count as lym's even with no prompt line recorded? Self-picked: yes, the first turn start past the prompt offset is the prompt's, as the carry-over already reads it.
