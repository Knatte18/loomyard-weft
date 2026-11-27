```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward, covering the failure and recovery ends of cards 1–9 and the next step each one names.

**Per refusal or stop: the next step it names, and whether that step can succeed**

| Card | Refusal or stop | Next step named | Progresses? |
|---|---|---|---|
| 1 | `StandingPassages` gives the zero `Standing` on a reader error | logged at Warn | not a refusal |
| 2 | `copyManifest` fails | logged at Warn, launch goes on | finding 3 |
| 3, 4 | manifest write fails | refused with the path | transient raw I/O, exempt (the prior dispute holds) |
| 6 | `PrepareEach` gets a `Prepare` error | names the engine and wraps `Prepare`'s own way forward | yes |
| 6 | `PrepareEach` finds no record | names `shuttleengine.WriteManifest` | yes |
| 6 | parity test finds a missing item | add a Codex delivery in `codexengine` | yes |
| 6 | a builder in `Seats` fails | none | finding 4 |
| 7 | producer scan finds an unlisted `Spec` literal | add the function to `producers`, or set no `Prompt` | yes |
| 7 | producer scan finds an unlisted `Table` literal | add the function to a `seatTables` entry's `Holders` | finding 2 |
| 8 | unknown seat | names the catalog's seats | yes |
| 8 | seat whose builder failed | relays the builder's error | finding 5 |
| 8 | missing item | add a Codex delivery; run no bench of the seat | yes |
| 8 | engine with unmet requirements | none | finding 1 |

**Findings**

1. **MEDIUM — card 8 (D10), the exit for unmet requirements names no next step, and it hits many seats.**
   `lym shuttle manifest` exits non-zero naming only the engine and its unmet requirements.
   Production fork settings show many catalog seats need forks:
   - `internal/loomengine/plan.go:133` and `internal/loomengine/discussiontable.go:158` set `ForkSubagents: forks`.
   - `internal/burlerengine/review.go:87` and `internal/burlerengine/fix.go:97` set it whenever a fan is configured.

   So the Plan-Write, Discussion and burler seats always fail the verb on Codex, with nothing telling the operator what to do.
   The message should say the seat cannot run on Codex until `codexengine` meets the requirement, so no Codex bench of it is possible.
   It should also say whether the printed manifests still serve D11's report.

2. **LOW — card 7, the way forward for an unlisted `seatengine.Table` literal can pass the scan without covering the seat.**
   The message offers only "add to … a `seatTables` entry's `Holders`".
   For a brand-new table builder that means filing it under an unrelated builder's entry, so its seats never reach `Seats` or the parity test.
   The step that succeeds is adding a new `seatTables` entry with `Builder` and `Build`, and the message should name it.

3. **LOW — card 2, the warning on a failed copy names no next step, and the failure repeats on every launch.**
   With a regular file at `&lt;root&gt;/manifests`, every later launch logs the same Warn and keeps no copy.
   D11's bench report opens with that copy, so a bench run loses its required input with no prompt to fix it.
   The Warn should name the path and say to remove whatever blocks it.

4. **LOW — card 6, `TestSeats_CodexManifestHoldsClaudes` "fails on any builder error" with no next step.**
   Unlike the missing-item failure, which names the `codexengine` fix, this one does not say what to change.
   It should say to fix the stand-in input `Build` writes for that builder in `seatTables`, since the builder ran on the stand-ins.

5. **LOW — card 8, a seat whose builder failed is refused with the builder's own error, whose way forward the operator cannot follow.**
   `Build` writes stand-in inputs into the verb's temporary directory.
   A builder's own way forward then points at a run's input, such as `PartsPlanTable`'s "running the Parts-Write row again" for an absent parts file.
   Those inputs are the verb's stand-ins, so that step cannot succeed.
   The refusal should name the catalog's `Build` stand-ins in `internal/shuttleparity` as what to fix.

**Outside this lens:** fork-requiring seats are compared on neither path.
The verb refuses them (finding 1), and the parity test only logs them.
That leaves the Plan-Write, Discussion and burler seats with no Codex parity check at all.
D1's acceptance criterion says "every seat", so this is coverage's to weigh.

**Tool calls**
- Searched `internal/seatengine` and the production code for `ForkSubagents`/`RequirementForks`, to learn which seats reach the verb's unmet-requirements exit; my context lacked which tables set forks.
- Listed every production `ForkSubagents:` setting, to confirm which seat builders set forks; the first search showed only literal `true` values.
