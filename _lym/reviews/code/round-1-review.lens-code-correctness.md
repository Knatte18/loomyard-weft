```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. For each function and branch the diff adds, I ran a concrete input through the code by hand and checked the result.

**Cases run**

- **`checkCreateCallerLater`**
  - Plan with `language: none` or `csharp`: `GoToolchain` is false, so the check returns empty. Correct.
  - Plan with no cards, or a card with no declarations: the loops never run and the check returns empty. Correct.
  - The reported case: card 1 creates `Level` alongside `level_test.go` and `template.yaml`, and card 2 `Uses` it while editing `internal/dobby#Tick`. One finding, on card 1, naming card 2. Correct.
  - Two later cards that can both call the function: the `break` after the first finding keeps it to one. The "first" card is the first in `plan.Cards` order, not the lowest number, and the parse path does not sort. Correct once card-numbering passes.
  - A later card that creates its own production function calling `F` and edits nothing else: that card cannot hold a caller, so it is skipped and card 1 gets no finding, even though card 1 fails its gate. This is a missed detection (F-2).
- **`isProductionFunctionDeclaration`**
  - A plain function: true. A method head `func (w *Watcher) Level() string` with handle `…#Watcher.Level`: the keyword reads `func` and the identifier `Level`, so true. A generic head `func Map[T any](x T) T`: keyword `func`, so true. All correct.
  - A `_test` unit, `main`, `init`, a head with `testing.TB`, and the name `TestX`: all false. Correct.
  - A production `FuzzyMatch`, `Testable`, `TestdataDir` or `Benchmarks`: `hasTestPrefix` matches the bare prefix, so the check skips them. It misses these and never wrongly refuses (F-3).
  - A head naming a `nettesting.Conn` parameter: the `testing.` substring match skips it. Same direction (F-3).
  - A helper declared on the package directory that lives in a package-internal `_test.go` file: true, and the plan text cannot tell otherwise. This is a wrong refusal (F-1).
  - A function in `internal/testkit/...`: true, but the guard exempts that package (F-1).
- **`cardCanHoldProductionCaller`**
  - A card whose only ref is the callee: false. An Edit group with `internal/dobby#Tick`: true. Delete, Rename, Move and Prosa groups are skipped. Create group refs hold the bare handle (`parse.go` `parseCreateField` refs), so `ref != callee` matches correctly. All correct.
- **`refCanHoldProductionCaller`**
  - Handle created on the card as `func`, `type` or `const`: false. As `var`: true. Created on another card: true. A `_test`-unit or `Test…` handle: false. All correct.
  - Path `internal/watch/template.yaml`: false. Paths `wire.go` and `internal/watch` (extensionless): true. `cmd/lym/testonlycode_linux_test.go`: false. All correct.
  - `Makefile`: true. This is the documented "uncertain counts" choice and only means a missed finding.
  - Package self glyph `internal/dobby#`: `UnitPath` gives `internal/dobby`, so true. Self glyph `internal/dobby_test#`: false. File self glyph `x/tick_test.go#`: false. `docs/x.md#`: false. All correct.
  - Member glyph `internal/dobby#Watcher.TestX`: `Name` is `TestX`, so false. Member glyph `internal/dobby#Tick`: true. A member glyph of a `_test.go` file unit: false. All correct.
  - Bare symbol `dobby.Tick`: `ParseRef` errors and returns true. This is the documented choice, and `bare-symbol-target` reports the ref separately.
- **`isTestHandle`**
  - `plan:x_test#h`: true. `plan:x#TestH`: true. A handle with no `#`: `handleUnit` and `HandleIdentifier` both fail, so false. Correct.
- **`declarationKeyword`**
  - `"  func(w *W) L()"`: `func`. `"type W struct"`: `type`. An empty string: returns empty and never matches. Correct.
- **`cardNamesRef`**
  - Exact match on `Targets` and `Uses`. Canonicalization rewrites every occurrence, so the spellings stay in step. Correct.
- **Execution paths**
  - Once the creating card is done, `dispatchFindings` drops findings on done cards (`planglyph.go`, `dispatchFindings`). A handle that `BindHandles` has rewritten after that batch cannot raise a finding on a pending card. Correct.

**Findings**

- **F-1 — MEDIUM, design.** Location: `internal/planparser/latercaller.go:72` (`isProductionFunctionDeclaration`) and `:31`.
  - The check refuses valid plans for functions the guard never checks.
  - Case: card 1 creates `plan:internal/watch#newFixture` -&gt; `func newFixture() *Watcher` and lists `internal/watch/fixture_test.go`. Card 2 has `Uses: plan:internal/watch#newFixture` and edits `internal/watch#Run` and `internal/watch/run_test.go`. Card 1 gets a create-caller-later finding, but the helper lives in a `_test.go` file and the guard never scans it.
  - The same happens for a function under `internal/testkit`, `internal/gitkit` or `internal/hubforge`. The guard exempts those packages (`cmd/lym/testonlycode_linux_test.go:35-39`), but the check classifies them as production.
  - The finding's second way forward, "list that caller on card 1", is wrong advice for a test helper.
  - The summary's Design decisions claims that requiring a later card that changes production code keeps a test-helper card silent. That holds only when the later card changes no production code.
  - Evidence: `latercaller.go:72-85`, `latercaller.go:42-47` and `cmd/lym/testonlycode_linux_test.go:35-39`.
- **F-2 — LOW, design.** Location: `internal/planparser/latercaller.go:46`.
  - Chain case: card 1 creates `A`, and card 2 creates `B`, which calls `A`, with `Uses A` and no other production ref.
  - Card 2 cannot hold a caller, so it is skipped, and card 1 gets no finding unless a later card that can hold a caller also names `A`.
  - Card 1 still fails its gate, so this shape of the reported defect goes undetected.
  - Evidence: `latercaller.go:42-48` and `refCanHoldProductionCaller`'s `func` branch at `latercaller.go:124-127`.
- **F-3 — LOW, design.** Location: `internal/planparser/latercaller.go:81` and `:84`.
  - The test-code tests are broader than Go's rule. `hasTestPrefix` matches a bare prefix (`vocabname.go:85-92`), so production `FuzzyMatch`, `Testable` and `TestdataDir` are skipped.
  - `strings.Contains(decl, "testing.")` also matches identifiers such as `nettesting.Conn`.
  - The effect is a silent miss, never a wrong refusal.

Tool calls:
1. `parse.go:610-680` (`parseTypeLabelCase`), to confirm that Create group refs hold the bare handle and whether cards are sorted. My context lacked the parse of the Create field.
