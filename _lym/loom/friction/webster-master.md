# Coordinating the board viewer batches

Batch 01-board-view-projection stopped as `waiting` after its first fork:
the plan adds production functions in cards 1–3 whose caller only arrives in card 6, in the next batch,
and the test-only-code guard fails on them.
The parent authorized temporary allowlist entries, and a second fork resumed the batch.
A plan that splits a function from its first caller across commits should name the temporary entries up front.
