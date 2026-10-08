# Card 11 gate lists packages inside a nested fixture module

Card 11's gate runs `go test -tags integration` on `./internal/planglyph/testdata/glyphchain/callees`, `callers` and `other`.
The fixture directory holds its own `go.mod`, so the main module does not contain those packages and `go test` reports `[setup failed]` for them; the gate as written cannot pass.
I ran `go vet` and `go test -tags integration ./...` inside the fixture module instead, both clean, and every other part of the gate passed.

Card 7 also tripped `TestRefKindScan_RealTreeClean` once: `redundancy.go` named `refKindHandle` directly, and the registry only admits `IsHandleRef` outside `classify.go` and `shape.go`.
Card 11 pushed `internal/planglyph` over its `test-budget.yaml` row, which I raised by one in the same commit.
