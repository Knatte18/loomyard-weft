# Implementing cards 7-9: an installer with its caller on the next card

Card 8 creates `fabricengine.InstallWorkspaceContent` and card 9 adds its only production caller,
so card 8's gate failed `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`.
I admitted the function in `testOnlyCodeAllowed` in card 8's commit, with `Pattern-Departure:` and `Loosened-Check:`
trailers, following the parent's earlier authorization for the same shape in batch 01, and card 9 removed the entry.
The plan should put a function and its first production caller on one card.
