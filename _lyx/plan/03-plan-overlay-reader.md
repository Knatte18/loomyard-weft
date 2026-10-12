# Card 3 — plan-overlay-reader

**Create:**
- `internal/planparser#Overlay`
- `internal/planparser#NewOverlay`
- `internal/planparser#Overlay.Parse`
- `internal/planparser#Overlay.readFile`
- `internal/planparser#Overlay.writeFile`
- `internal/planparser#ErrOverlayName`
- `internal/planparser#ErrOverlayFileMissing`
- `internal/planparser#Plan.ReadFile`
- `internal/planparser#Plan.RewriteRefs`
- `internal/planparser#Plan.Reparse`
- `internal/planparser#rewriteRefsIn`
- `internal/planparser_test#TestOverlay`

**Edit:**
- `internal/planparser#Plan`
- `internal/planparser#RewriteRefs`
- `internal/planparser#rewriteCardFile` -> `func rewriteCardFile(plan *Plan, c Card, subs map[string]string, read func(name string) ([]byte, error), write func(name string, data []byte) error) error`
- `internal/planparser#cardHeadingNumber` -> `func cardHeadingNumber(plan *Plan, c Card) (int, bool)`
- `internal/planparser#checkCardNumbering`
- `internal/planparser/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Give planparser a plan source that answers from proposed contents first and disk second, so a plan can be parsed, checked and canonicalized in memory without a byte written (Decision 5, the planparser half).

`Overlay`, in a new file `internal/planparser/overlay.go`, holds a plan directory and proposed full contents keyed by plan file name.
`NewOverlay` accepts only `00-overview.md` and the card file names (`NN-<slug>.md`) the Card Index of the overview on disk lists, read through the same overview parse `ParsePlanFrom` runs; any other name — one with a path separator or `..`, a name not ending in `.md`, `amendments.md`, or an `.md` file on disk the Card Index does not list — is refused with `ErrOverlayName`, naming the name, and a Card Index name whose file is absent on disk is refused with `ErrOverlayFileMissing`.
An overview on disk that does not parse is returned as its parse error, which the verb already reports as a plan that does not parse.
`Overlay.readFile` answers an overlaid name from memory and any other from `planDir` on disk; `Overlay.writeFile` stores into memory only; `Overlay.Parse` is `ParsePlanFrom(planDir, o.readFile)` with the overlay recorded on the returned `Plan`.

`Plan` gains an unexported field holding its overlay, nil for a plan `ParsePlan` or `ParsePlanFrom` built, so `reflect.DeepEqual` between those two stays true.
`Plan.ReadFile` reads a plan file through the overlay when set, else from `Dir` on disk.
`Plan.RewriteRefs` substitutes through the overlay in memory when set, else is `RewriteRefs(Dir, subs)` on disk; `Plan.Reparse` is `Overlay.Parse` when set, else `ParsePlan(Dir)`.
`RewriteRefs` becomes `rewriteRefsIn` over `os.ReadFile` and `os.WriteFile` joined with `planDir`, and `rewriteRefsIn` parses through `read` and rewrites each card through `rewriteCardFile`, which reads and writes through the two functions it is given; so the in-memory substitution is the on-disk rewrite's own code, with its byte-identical and lexeme rules (Decision 5's bound).
`cardHeadingNumber` reads the card file through `plan.ReadFile`, so `card-numbering` judges an overlaid card's heading; `checkCardNumbering` is its caller.
The directory listing `checkIndexFileConsistency` reads stays the disk's, which an overlay cannot change because it adds and removes no file.

`doc.go` names `Overlay` beside the write paths: it is a read path that writes no `_lyx/plan/` byte.
`TestOverlay` is a new top-level test, so `internal/planparser`'s row in `test-budget.yaml` rises by one.
Its table pins: an overlaid card is parsed from memory while the other cards come from disk; each refused name (`sub/01-x.md`, `../00-overview.md`, `notes.txt`, `amendments.md`, an orphan `99-orphan.md` written on disk) is `ErrOverlayName`, and a listed card whose file was removed from disk is `ErrOverlayFileMissing`; `Plan.RewriteRefs` on an overlaid plan leaves every plan file on disk byte-identical while `Plan.Reparse` returns the substituted ref; `ValidateFormat` over an overlaid card whose heading number differs from its Card Index number reports `card-numbering`.

**ImpactSummary:** additive in planparser; `rewriteCardFile`'s one caller and `cardHeadingNumber`'s one caller are in this card, and `RewriteRefs`' exported signature and behavior stay as they are for its three callers.

**Commit:** 3: planparser parses and rewrites a plan through an in-memory overlay
