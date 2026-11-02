```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I re-ran the searches behind F1, F7 and F2/F3 and found no gaps.

- **F1 ("card" literals):** `config.go` has no bare `"card"` left. The remaining `"card"` strings are the `cardGateKey` and `scopeCard` declarations, plus the `goReadGates` slice (`tiers.go:103`). That slice is a list of gate names, not a key check in `decodeGates`, so it is outside the review's premise. The bare `"batch"` literals in `config.go` are untouched and were not named by F1.
- **F7 (goto sites):** the three move sites the review named now go through `c.gotoGuardEndingFixers` (`drivewiring.go:281`, `arm.go:464`, `arbiterconsume.go:28`). The only other guard-building sites I found are validation guards: `drivewiring.go:208` (the tackler's `GotoGuard`), `arbiter.go:140` (the arbiter call's `GotoGuard`, checked at `shedarbiter/call.go:339`) and `battencli/drive.go:487` (`haltedRowGuard`). The three move sites write the move; the validation sites only decide whether a verdict's goto is admissible. Leaving them unwrapped matches the review's list of three. The new test covers the `arm.go` site only. The `drivewiring.go` and `arbiterconsume.go` sites are covered only through the shared helper.
- **F2/F3 (callers):** every `waitOnly` and `waitThenRun` caller passes the new flag (`rowgate.go:85`, `86`, `104`, `132`, `144`, `196`), and there are no other callers in the tree.
- **F9 (`"verify"` literals):** the only bare `"verify"` left in `internal/loomcli` is the `darnGateEntry` declaration, plus an unrelated `formatWaitNote` string at `reviewstatus.go:124`.

Calls made (3 of 6):
1. `grep '"card"' gateslot/*.go`: the F1 search, since my context only held the three sites named by line.
2. `grep` over `internal` for `fixerEndingGuard(`, `endRunFixers` and `GotoGuard`: the F7 site search, to find sites the review did not name.
3. `grep` over `internal` for `waitThenRun(` and `waitOnly(`, plus `"verify"` in `loomcli`: the F2/F3 caller search and the F9 literal search.

I left no finding unchecked.
