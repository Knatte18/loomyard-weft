# Plan format has no way to insert a card or remove a card file

`plan-card-insert` · Note · enhancement

Imported from [issue #508](https://github.com/Knatte18/loomyard/issues/508).

Plan cards are numbered contiguously and each card file name is tied to its number.
A Plan-Review fix that needed one new card ahead of card 8 therefore renumbered cards 8 to 19, twelve files.

The fixer rules allow file changes only through Edit or Write and name no way to rename or remove a file.
The fixer rewrote each card under its new name and removed the old files with `rm`, which the rules neither allow nor forbid.

This is abnormal: a routine plan edit forces a fixer to guess at an unstated rule and do a bulk, error-prone rewrite.

What to do, either of:
- add a plan verb that inserts a card and renumbers the rest;
- state a rule for removing a file inside the fixer's write surface.

Provenance: task `driver-context`; note `burler-plan-r1.md`.
