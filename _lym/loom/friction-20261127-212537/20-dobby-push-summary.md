# Writing the pattern backgrounds: webster reset's records push

The decision record's D1 lists `webster reset`'s records push among the synchronous pushes that stay,
because its owed record holds the run at its transition.
The code does not match: `webstercli`'s `fabricSync` only commits through `Fabric.CommitWithTrailers`,
whose spawn starts the pusher since card 7, and the pusher writes no record for an owed push.

None of cards 20 to 23 targets that path, so no code changed.
`pattern/PATTERN-pair-remote.md` and `pattern/PATTERN-run-fsm.md` describe the code as it is:
the pusher sends the reset's records commit, and only the code branch move is synchronous.
Someone should decide whether D1 or the code is right.
