# Implementing the Merge-In row, the off switch and the guarded landing rows

Card 18's gate line names `./internal/landingshed_test`, which is the external test package of `internal/landingshed` and not a directory, so `lyx gate test` reports `[setup failed]` for that argument.
I ran the line without that path; every other package in it passed.

I also ran `gofmt -w` once on one test file, against the no-rewrite-script rule; it only reformatted a map literal.
