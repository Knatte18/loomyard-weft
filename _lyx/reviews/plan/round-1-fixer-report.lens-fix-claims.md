```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked whether the sentences the fix added or rewrote hold against the code at HEAD. One claim fails.

**[BLOCKING] Card 39 (`39-adopt-refusals.md`), the live-pair way forward and its new `TestAdopt` row (fixes review F16).**
The changed text says `lyx tender pause <slug>` makes `LivePair` read not live: "the way forward names first the step that ends the pair's driver, `lyx tender pause <slug>`, under whose pause entry `LivePair` reads not live". The new test row adds "with the fake world's pause covering the parent, so `LivePair` reads not live, that call passes".
- **Summary:** a pause scoped to one slug never makes adopt's `LivePair` read not live, so the named sequence is refused again at step 1 as typed. That is the same defect F16 raised, still unresolved.
- **Evidence, code:** `internal/adoptcli/deps.go:45` wires `options.HubPaused = hubOnlyPaused(geometry.RecordPath)`. `hubOnlyPaused` (`deps.go:114-126`) returns true only for a live entry whose `Scope == pauserecord.ScopeHub`.
- **Evidence, doc:** `internal/adoptcli/doc.go:19` says: "covers a slug only by a live hub entry, so a pair whose strands are alive under any slug-scope pause, an owner pause included, reads live."
- **Evidence, test:** the new `TestAdopt` row passes only because the fake world counts a slug pause, which production does not.
- **Possible fixes, from the code:**
  - A hub-wide pause, `lyx tender pause` with no slug, does write a hub entry.
  - `takeOverSlug` (`adopt.go:292-317`) passes a live pair under `--keep-local`, and its own way forward offers exactly that for "a live driver".

The other claims I checked hold:
- **Card 28:** `registerSeededRun`'s `windowFlag` check is at `arm.go:296`. `TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree` uses `openTerminalFlag` (`wire_seams_test.go:256`).
- **Card 24:** the rule table's budget and precondition hand-backs are in `driverules.go:239-254`.
- **Card 15:** `lyx loom resume` spawns a Go driver for a halted go-seeded run with no driver (`resume.go` Long).
- **Card 27:** `lock.TryAcquireWriteLock` exists. Its go-test guard is `testing.Testing()`, from the standard library, so `internal/lock` is the card's one new import (`logger` is already on card 4's list).
- **Card 36:** `segmentcolor.Coordinator` exists.
- **Card 14:** the owner re-check at `innerrun_pause.go:65`, which runs whenever `ownedBy` stands, covered or not.

Tool calls:
1. Read adoptcli's `HubPaused` wiring, the `loom resume` help, `registerSeededRun`, the `wire_seams_test` flag use and `arbiterSkills`. My context lacked the adopt pause seam's scope and those bodies.
2. Read existing `Skills` lists, the `driverules` hand-back helpers, the go-test spawn guard and the `hubOnlyPaused` body. My context lacked them to check the card 24, 27, 36 and 39 claims.
