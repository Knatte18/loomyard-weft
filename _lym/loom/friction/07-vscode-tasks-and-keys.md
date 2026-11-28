# Implementing cards 7-9: card 8 could not pass its gate alone

Card 8 creates `fabricengine.InstallWorkspaceContent`, whose first `main` caller is card 9's wiring in `newRoot`,
so card 8's gate failed `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`.
I followed the way the run's parent chose for the same guard in the first batch:
a temporary `testOnlyCodeAllowed` entry in card 8 with a `Pattern-Departure:` trailer, removed in card 9.
