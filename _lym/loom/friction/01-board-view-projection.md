# Batch 01: new exported functions have no caller from main until a later batch

Card 1 adds `BoardRevision` and `Board.View`, but their callers arrive in cards 4 and 5, and `main` reaches them only in card 6.
The card gate's tier1 guard step runs `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`, which fails on both.
`Board.Expecting` (card 2) and `ViewerPage` (card 3) will fail it the same way, and the plan names no way through.
