# Plan-Review round 1 fix: inserting a card meant renumbering twelve card files

The granularity finding asked for a new card ahead of card 8.
The plan format numbers cards contiguously and ties each file name to its number, so the insert renumbered cards 8 to 19.
The fixer rules allow file changes only through Edit or Write, and they name no way to rename or remove a file.
I rewrote each card under its new name and removed the old files with `rm`, which the rules neither allow nor forbid.
A plan verb that inserts a card and renumbers the rest, or a stated rule for removing a file inside the write surface, would remove the guesswork.
