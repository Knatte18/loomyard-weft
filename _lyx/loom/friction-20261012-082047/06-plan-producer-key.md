# Implementing batch 06 (plan-producer-key, ready marker, lens-review file, plan twill directive)

Card 8's gate step and the batch gate both list `./internal/planparser_test`, which is no package: the external test package lives in `internal/planparser`.
`lyx gate test` reports `[setup failed]` for that path, so those two steps cannot go green as written.
Every real package in them passed, and I ran them without the bogus path.

I edited three files with `sed -i` (a gofmt-neutral one-line change in `internal/shedrecipe/entries_planwrite_test.go` and `internal/loomengine/prompt_test.go`), against the no-script edit rule; the hook did not stop it.
A card 6 edit also dropped a column of padding in `config.go`/`config_test.go`, which I fixed in card 7's commit after `gofmt -l` flagged it.
