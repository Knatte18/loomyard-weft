# Burler code round 1 fix: matching a killed package

F2 needs to tell the killed binary's own package failure from another's.
The review proposed the killed binary's directory, and the first fix followed it with a scripted test that passed.
A self-check fork then showed that a test's chdir moves that directory.

The next idea, the package failure's own `signal: killed` line, also failed: `go test -json` files that line under the running test, so the package failure carries no line.
Only a real `go test` run showed it.
A scripted event stream cannot pin what `go test` prints, so a finding that rests on that output needs a real run first.
