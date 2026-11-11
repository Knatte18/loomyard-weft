```yaml
lens: forced-edits
outcome: reported
```

Lens forced-edits: what each card's change forces outside the members it names.

- **LOW** — Card 1 (commit subject). Card 1 changes the owner write's commit subject: with a replace set it appends ` replacing <names>`, and with no slugs and no `Prime` it drops `for <subjects>`. `internal/boardengine/doc.go:174` still states the commit as `"board: owner <machine> for <slugs>"`. Card 1 lists `doc.go`, but its Intent only adds a replace-set paragraph and never names this line. Fix: name the rewrite of line 174 in the Intent.
- **LOW** — Card 1 (`OwnerWrite` field docs). With `Replace` set, the prime is written "whatever `req.Prime` says", and `Recorded` gains slugs that are not in `req.Slugs`. The `Recorded` field doc on `internal/boardengine#OwnerWrite` (`internal/boardengine/ownerset.go:62`) reads "lists the slugs, and OwnerWritePrime when asked", which becomes wrong. `internal/boardengine#OwnerWrite` is on no card's `Edit`. The comment on `internal/boardengine#writeOwners` also says it "sets the owners req asks for"; that member is on card 1, but the Intent does not name the comment. Fix: add `internal/boardengine#OwnerWrite` to card 1's Edit, with its `Recorded` doc naming the replace set.
- **LOW** — Card 4 (pull seam mutations directory). The Intent fills `pull` with `fabricengine.NewMutations(hub)` and says this is "as `SetOwners` builds them". `SetOwners` builds `fabricengine.NewMutations(filepath.Dir(boardDir))` (`internal/boardengine/ownerset.go:113`), and `boardDir` is `fabricengine.BoardDir(hub)`, which is not the Bolt checkout root. The two operands differ, so the card either misstates its source or records the mutations in a different directory. Fix: use `filepath.Dir(boardDir)` as `SetOwners` does, or drop the "as `SetOwners` builds them" claim.
- **LOW** — Card 4 (cwd hub detection versus `PATTERN-cli-cobra` sole wiring). `hubsToSweep` finds the hub of the working directory with `lymcwd.ResolveWorktree` and `preflight.BoardLymPresent`. That rebuilds the hub-or-standalone split that `internal/preflight#ResolveMode` owns (`internal/preflight/predicates.go:96-119`). `pattern/PATTERN-cli-cobra.md`'s "Sole wiring" section makes `internal/cliwire` and `ResolveMode` the owners of that resolution. The card states no departure, and `## Shared Decisions` records none. Fix: either reason the departure in Shared Decisions (D5 runs from a wired hub's subdirectory, where `ResolveMode` refuses), or route through the resolver.

No unlisted forced edits found in these places:
- Every glyph the cards list resolved through `lym quarry resolve`.
- `internal/shellguard/rows.go` holds no per-module row.
- The other module lists that name hibernate are `docs/benchmarks/test-suite-weight.md`, a measurement report, and `contracts/orchrole/role-hibernate.md`, which is specific to hibernate.
- Card 3 covers the remaining module-list pins: `cmd/lym/main.go`'s `Long`, `seamsignature_test.go`, `excludedModules`, `docs/overview.md` and the `PATTERN-cli-cobra` deviations.

Tool calls:
- A grep for the owner-write commit subject and the "only `lym adopt`" or owner-setter prose. My context lacked the readers of those strings outside `boardengine/doc.go`.
- A grep for module lists that name `hibernate`. My context lacked the full set of module-list pins.
- `lym quarry resolve`, run twice (the first run's filter printed nothing), over every glyph the cards list. My context lacked confirmation that those glyphs resolve.
- A grep for scans guarding `lymcwd.ResolveWorktree` or `preflight` callers. My context lacked whether a test forbids a `*cli` package from resolving the hub itself.
- A grep of `internal/shellguard/rows.go` and the benchmark doc for module rows. My context lacked whether a guard table needs a `machine` row.
