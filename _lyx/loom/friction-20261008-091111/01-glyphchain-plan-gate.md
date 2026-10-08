# Implementing batch 01 (cards 1-6): glyph-chain fixture and containment retirement

Card 1's gate runs `go test -tags integration` on `./internal/planglyph/testdata/glyphchain/shapes`, `dirname` and `cmd/tool`.
The same card creates `testdata/glyphchain/go.mod`, which makes the fixture its own module, so Go answers `main module does not contain package ... [setup failed]` for those three paths and the `&&` chain cannot pass as written.
I ran the fixture's own tests from inside its module instead (`go vet` and `go test -tags integration ./...` there, both clean) and treated every other part of the gate as binding.

The test-budget guard in `cmd/lyx/testdata/test-budget.yaml` needed `internal/planglyph` raised twice (cards 1 and 2); that file is on no card's target list.
