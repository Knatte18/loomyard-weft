# Card 16: the creel-core board entry cannot be cut to a pointer from inside its own run

Card 16 asks for the `creel-core` board entry's body to be cut to a pointer at `internal/creelengine`'s package doc through `lyx board upsert`.
The board refuses it: the entry is held by this very run (status `running · Webster`) and takes no scope edits.
Clearing the status is the orch's call, and only once no run holds the entry, so the fork did not work around it.
The card should name that step for the landing or the orch instead of for the implementer.
