# Card 3 — planparser-parse-from

**Edit:**
- `internal/planparser/parse.go`
- `internal/planparser/parse_test.go`

**Intent:**
Let the sole plan parser read a plan from somewhere other than the working tree, so the rework row can parse the plan as committed at HEAD without writing plan bytes anywhere (Planparser Sole-Parser Invariant).
Add `ParsePlanFrom(planDir string, read func(name string) ([]byte, error)) (*Plan, error)`:
`read` returns the contents of the named top-level plan file (`00-overview.md`, `NN-<slug>.md`), and an error wrapping `fs.ErrNotExist` when the file is absent.
`ParsePlan(planDir)` becomes `ParsePlanFrom(planDir, <os.ReadFile over filepath.Join(planDir, name)>)`, and `parseCardFile` reads through the same closure;
the not-found checks switch from `os.IsNotExist` to `errors.Is(err, fs.ErrNotExist)` so both readers produce the same "not found" errors.
`Plan.Dir` stays the told `planDir` in both forms.
Add tests: `ParsePlanFrom` over an in-memory map reader parses to the same `Plan` as `ParsePlan` over the same files on disk, and a map missing a card file yields the "card file not found" error.

**ImpactSummary:** ParsePlan keeps its signature and behaviour; every existing caller is unaffected.
