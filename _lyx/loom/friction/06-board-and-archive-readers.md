# Batch 06: gate steps naming `_test` directories

Card 6 and card 8's gate steps list `./internal/boardengine_test`, `./internal/fabricengine_test` and `./internal/gitrepo_test`, which are external test packages, not directories; `lyx gate test` fails them with `directory not found`.
I ran the real package directories instead, which cover the external test packages.

Card 8 adds `BlameLines` to the `r.runChecked` method set, which `cmd/lyx/gitrepoboundary_test.go` pins by name; the plan names no card edit for that file, so it surfaced only when I ran `./cmd/lyx` by hand.
