# Publish verify failed twice on a shedverbs loop-wait flake outside this task's change

Two consecutive Merge-In verifies failed in `internal/shedverbs`'s integration tier, on different wait-world tests, each with the waiter reporting `loop … exited before it recorded itself`.
Neither reproduced in isolation, under `-race`, under `-count=300` or beside heavy integration packages, which cost several runs.
The likely cause was the wait-world fixture's 20 ms `LockGrace`, which the waiter's free-lock probe can exhaust under scheduling stalls; the fix gives the fixture the production grace.
