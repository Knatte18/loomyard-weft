# Building the card 15 replay corpus from real runs

Card 15 sends the implementer to source data that sits behind the fabric-reference audit.
The hub orch run's events file lives physically under the weft checkout, and the earlier attempt at this batch was rejected for reading run files through `*-weft` paths.
This recovery read the same files read-only through the worktrees' own `.lyx` links, which name no weft path.
The card does not say which route is allowed, so a recovery strand has to rediscover it.

The card also describes three cases more precisely than the real runs support.
The continuing session of the `continued-in` case never wrote a transcript, the Plan-Write ask was torn down before its answered turn ended, and the API-error fork was followed by a re-fork rather than a done or held turn end.
Each needed a transcript search to establish what really happened before its case could be written as a correction.
