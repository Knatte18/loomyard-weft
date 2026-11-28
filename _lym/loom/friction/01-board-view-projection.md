# Implementing batch 01: card gates cannot pass the test-only-code guard

Cards 1-4 add `boardengine` functions (`Board.View`, `BoardRevision`, `ViewerPage`, `Board.Expecting`)
whose only production callers are `internal/boardserve`, and no `main` package imports that until card 6, in the next batch.
`TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` runs as a guard in every card gate, so card 1 already fails it.
The plan says nothing about this, and the guard's allowlist criteria do not cover a function that is waiting for its caller.
