# Burler fix, code round 1: reed client reads and shellguard rows

F3 asked for an exception in a shellguard row, but a row's patterns can only be ANDed and `grep -E` has no negation.
The first version spelled out "any word but `--system`" with quote and escape parsing, and failed open on input it could not parse; the self-check caught it and a second version replaced it.

The claude settings golden has no update flag, so each steer or pattern change needed a temporary test file to render it.
