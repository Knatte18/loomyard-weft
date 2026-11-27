```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each change in the diff to the decision that asks for it.

**Per change → decision**

| Change | Traces to |
|---|---|
| `scankit` `AssertNoImports` and `TestBannedImports` | D1 leaf packages (card 1) |
| `fabriclayout` package, leaf tests, classification and `ScopedPathspec` move | D1, D9.1, board goal (card 2) |
| Hub paths, ref names, mutation kinds and record, origin, push, verb-result and read types moved, callers retargeted | D1, D7, D9.1 (cards 3–8, 10, 11) |
| `failedPushSide*` constants replaced by `fabriclayout.Repo*` | forced by D1's no-alias rule (card 9) |
| Method-to-function and export renames (`branch*WayForward`, `AdoptTarget`, `PairSlug`, `IsZero`) | forced by the type moves (cards 7, 8) |
| `rec.Extend` replaced by `MutationsOf(append(...))`, in `fabriccli` and the reconcile files | forced by the record/recorder split (card 6) |
| `moveLockFile` and `originRecordLockFileName` redeclared | forced by their neighbours leaving a const block |
| `cmd/lym` result-type table, `isMutationRecordEmbed`, watch patterns | forced (cards 6, 8, 10, 11) |
| `geometryTokenOwners`, `weftnameImportOwners`, `fabricvocab` owners, import allowlists | D7 and the board goal; loosened allowlists are forced |
| `PATTERN.md` and `pattern/` edits | declared PATTERN changes, D7, forced renames |
| `docs/overview.md`, `docs/logging-inventory.md` line numbers, `doc.go` and comment renames | forced |
| `docs/research/*` and `tools/tokencount` | treated as forced retargets: my filter showed no change beyond qualifiers, and I did not read the pieces individually |
| `fabriccore` types | D1, D2, D3, D11 (card 12) |
| `fabrickit` and `TestEnforcement_FabrickitLeafOnly` | D8 (card 13) |
| `executor.go` and its tests | D3, D9.1 (card 14) |
| `snapshotread.go`, `export_test.go` accessors and its tests | D2, D6 (card 15) |
| `testOnlyCodeAllowed` entries and doc comment | added entry of 2026-11-27 on production code only a later part calls (cards 13–15) |
| Engine test deletions and trims | D5, D6, both added entries on test shrink (cards 2–4, 15) |

I found no unrequested change.

**Per decision → changes**

| Decision | Built by |
|---|---|
| D1 | layout, core and leaf tests |
| D2 | snapshot types and reader; seam facts are deferred by the plan's snapshot-fields decision |
| D3 | action and outcome types, executor; backoff delays deferred to part 2 by card 12 |
| D5 | retarget with no behaviour change found in the non-test hunks I filtered |
| D6 | reader seam test (snapshot half) and git-fallback test |
| D7 | retargets, owner maps, allowlists |
| D8 | `fabrickit` |
| D9 | the part 1 content above |
| D11 | "snapshot" and "action list" wording in the core and executor |
| Added: `push-retry-not-halt` waits for the chain | none needed in the diff (a board matter) |
| Added: `hubforge`, `boardkit`, `envkit` keep import sets | holds; no change beyond retargets seen |
| Added: secret-path refusal as snapshot fact | none; it belongs to the later part that moves `Commit` |
| Added: process-installed seams | none; no seam is installed in this part |
| Added: restructure cuts test code | cards 2–4 and 15, only marginally (finding 1) |
| Added: later-part-only production code | the allowlist entries |
| Added: this part shrinks in net | met: 58594 → 58200 lines |

**Findings**

1. **MEDIUM, class `scope`.** The part's fabric test code barely shrinks.
   - **Location:** the `_test.go` files of `internal/fabricengine`, `internal/fabriclayout`, `internal/fabriccore` and `internal/testkit/fabrickit`; commit `f4fd75355` states the totals.
   - **Measured:** base `6fbe0ff7a` has 58594 lines, all in `fabricengine`. HEAD has 58200: `fabricengine` 57286, `fabriclayout` 728, `fabriccore` 74, `fabrickit` 112. That is 394 lines down, 0.67%.
   - **Rule:** the added entry "The restructure cuts fabric's test code, it does not only move it" says every part shrinks the fabric test code it touches, and "review raises a part whose test code grows or barely shrinks". The acceptance criterion repeats "each part's fabric test code shrinks".
   - **Not a defence:** the later entry "This part's fabric test code shrinks in net" is met, but it adds a floor and does not withdraw the "barely shrinks" clause.
   - **Evidence:** my `git ls-tree` and `wc -l` measurement at both revisions; `_lym/discussion/decision-record.md`, the two added entries.

**Not verified (not a finding).** A name scan shows `SecretPathsKey`, `WarpJunction`, `OwnerArchivePair`, `HandOverRequest` and `TakeOverRequest` still declared in the engine and referenced as `fabricengine.X`. The scan also counts the engine's own external tests, and I had no calls left to check whether any file outside fabric uses them or whether their fields are plain data. The plan's "what the layout takes" decision would require moving any that fit.

**Tool calls**

1. `cat` of plan cards 4–11: the manifest held only cards 2, 3 and 12–15.
2. Filtered `git diff` for hunks that are more than a qualifier rename: most diff pieces were not in context.
3. Read of call 1's saved output: it was truncated to a preview.
4. Filtered view of call 2's output plus per-revision test line counts: the output was truncated, and the totals needed independent measurement.
5. Second filtered view of call 2's output: call 4's view was cut off before the files outside the engine's new code.
6. Grep for aliases and for names still declared in the engine: needed to check D1 and D7, which no context file showed.
