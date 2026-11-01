# A card that retires a name sweeps the whole tree, tests included, first

`retired-name-sweep` · Task · enhancement, webster, stencil · running · Code-Bouncer

## Seen (orch, 2026-11-01)

In `run-verbs-shared`, the fork for card `06-tapestry-config-module` removed the `loom.plan_*` keys, but its first grep left out `_test` files.
Stale fixtures carrying the old keys then failed across about ten packages, found one gate failure at a time at about three turns a file: 69 gate-loop turns of the fork's 165.
Another fork read and edited each file of one grep hit list one at a time over some twenty turns.

## Idea

The Webster implementer stencil gains a sweep step: when a card removes or renames a name or key, the fork greps the whole tree for every retired name, test files and test data included, before its first edit, reads every hit's span in one message, and edits every hit in one message.
