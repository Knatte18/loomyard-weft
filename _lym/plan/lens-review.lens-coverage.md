```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked whether every decision, acceptance criterion and card branch has a planned test row that would fail if it broke.

1. **BLOCKING** — PATTERN changes, the overview's Shared Decision `pattern-headroom`, cards 4 and 12. The plan says outright that its own fenced lines push `PATTERN.md` past the headroom line. So `pattern-headroom` and `pattern-card-headroom` fail at the plan gate as written, and the fix is left as "the parent's to take". The Step 1 route covers a design call like this: ask the parent to record a `changes` decision, and plan on it only once the record carries it. Evidence: overview `## Shared Decisions` "Decision: pattern-headroom" ("PATTERN.md stands at 15560 bytes against a 15564-byte headroom line, so the projected overview grows past it").

2. **BLOCKING** — D6 `depends_on` rule, card 12. The card admits a `depends_on` on a `board_checked` entry gone from the board only "with `Filing` true". D6 says outright that `depends_on` "names only parts, entries on the board, or `board_checked` entries since gone from it". So the gate and `raddle-approve`, which run with `filing` false (cards 19 and 20), refuse a proposal D6 admits whenever a weighed entry is pruned between the agent's read and its propose or approve. No row covers the non-filing case. Evidence: card 12 Intent, `depends-unknown` bullet; decision record D6, fifth bullet.

3. **MEDIUM** — D9 filing level, cards 16, 22 and 23. `RaddleFiling.Parent` and `Entry` take the level from `hubgeom.ParentOf`. That returns the nearest open beam ancestor (`boardengine.ParentBeam`: `Kind == KindProject && Beam`), not the Sketch's own project. A Sketch inside a non-beam grouping project would have its parts filed into the beam above it, or at the top level, instead of "at the Sketch's own project level" (D9 step 3). That is the near operand, not the named one. No row covers a Sketch whose `Project` differs from its beam. Evidence: `internal/hubgeom#ParentOf` and `internal/boardengine#ParentBeam` (parentbeam.go:7-38); card 16 `RaddleFiling` "Parent".

4. **MEDIUM** — D7 "current Raddle-Write session", card 21. The session number rises only when a Raddle-File return is taken up. A new Raddle-Write agent session after a halt or crash resume, with no return, keeps the old number. `raddle-approve` would then pass on the prior session's propose, and no waiting notice goes out. D7's bound requires the notice "in every session that approves". No row covers it. Evidence: card 21 Intent, second bullet; card 19 `session` field.

5. **MEDIUM** — D8 "returns … in a new session", cards 21 and 23. `NewRaddleWrite` raises `session` but never says how the inner `NewMultiLLMProducerSourced` is made to start a fresh agent session rather than re-attach to the previous seat's live or recorded one. Exploration found that nothing in shed's types makes a bounce start a fresh session. `TestRaddleWrite` uses a stub inner producer, so a real re-attach would pass every planned row. Evidence: card 21 `TestRaddleWrite` rows; parent exploration of shedengine outcomes ("a RaddleWrite producer would have to start one itself").

6. **MEDIUM** — D8 return from inside the board write, card 22. "The write's refusal there is a return like the pre-check's", but `FileRaddle` returns `Check`'s error "untouched" (card 16). No sentinel error tells a re-check refusal from an I/O failure of the write. And no row drives `File` returning `Check`'s refusal into a return, as opposed to the pre-check failure row. Evidence: card 16 `FileRaddle` first bullet; card 22 step 3 bullet and its rows.

7. **MEDIUM** — D10 replay "refuses a filed entry's add", card 15. The rows cover only a *lost* add (origin already holds the slug). The criterion names a replay that *refuses* the add (`refuse` via `validateWrite` or a dependency, `addEntry`), and no row has a refused add setting the store aside. Evidence: card 15 `TestFabricReplay` rows; decision record D10 criterion "a replay that refuses a filed entry's add sets aside the store's files".

8. **MEDIUM** — D2 batten guard, card 24. The criterion names a slug the store holds "as another Sketch's, with another `created` stamp". The rows cover another Sketch and a filed slug, but not the entry's own slug under a different recorded `created` stamp, such as a re-created entry. That branch of `raddleStartRefusal` goes unreached. Evidence: card 24 `TestStartGuard_Refuse` rows; acceptance criterion D2, second bullet.

9. **MEDIUM** — D12 "every agent session a run spawns", card 5. The hook is installed only on `loomCLI.wire`'s runner and `gatecli#newStrandRig`. Exploration shows other runners built in a run's processes, `internal/loomcli/looparm.go:122` (the loop arm, which carries the step watchdog) and `internal/battencli/drive.go`, and a session started through either writes no usage file. No row or Intent line accounts for them. Evidence: the grep of `shuttleengine.NewRunner` sites in the exploration.

10. **MEDIUM** — D13 "landed runs", card 8. `ComputeCost` selects "every run whose archive tag falls in the window, whatever its recipe". That is archived runs, not landed ones, so cancelled runs and runs torn down unlanded enter their cell's means. No row covers a cancelled, unlanded tapestry run being excluded. Evidence: card 8 Intent, first line; decision record D13 "reads the usage records of landed runs".

11. **MEDIUM** — D13 branches, card 8. Two branches the Intent states have no row. "A fit needs two sized runs with distinct sizes, else it is absent" is one. "A run the timing report does not time has unknown hours and enters the dollars alone" is the other, and it covers every motif and raddle run, since `ComputeTiming`'s `runRecipes` holds only tapestry and darn. Evidence: card 8 `TestComputeCost` rows; `internal/reportengine/runs.go:24`.

12. **MEDIUM** — D9 part records, card 22. The project's record and the `tapestry` shape's one part record are "the Sketch's record whole", but no row asserts the content of any record step 2 writes, nor the Goal and Scope rendered from the contract. `TestSplicePart` covers only the splice function. Evidence: card 22 `TestRaddleFile` rows; acceptance criterion D9 "each part record … holds exactly…".

13. **MEDIUM** — D1 batten child recipe, card 23. `childRecipeSupported` and `unsupportedChildRecipeRefusal` change, but no row asserts batten admits `raddle`. The criterion "a board task with recipe `raddle` runs through batten" rests on the recipe-loader row alone. Evidence: card 23 test list.

14. **LOW** — D2, card 13. No row covers a plain promote, or a non-raddle promote, of a discussed note a note write had stripped of its label. The criterion says "whether the note carries the label or not". Evidence: card 13 `TestDiscussedPromotion` rows.

15. **LOW** — D7, card 20. The "first session after a return" case of `raddle-approve`'s no-propose refusal is not named as a row (`proposed_session` 1, `session` 2). Evidence: card 20 `TestRaddleApproveVerb` rows.

16. **LOW** — Unreached branches stated in Intents:
    - Card 3: a session with no session id is not reported.
    - Card 6: a landing with no recorded tip skips the size record, and a board or profile read failure is logged at Warn.
    - Card 18: an entry flagged discussed whose record file is missing falls back to the body.
    - Card 21: an `AutoApprove` read error.
    - Card 25: a `Notify` failure after `HandOff` found a line.

17. **LOW** — D3, card 18. "For darn, raddle and pawl alike" is tested only at producer level. No wiring row shows pawl's run reaches Record-Write's stored branch for a raddle-filed project. Evidence: card 18 rows.

18. **LOW** — Card 14. Reading the store under every write's lock means one unreadable `store.json` fails every board write. No row covers that outcome, and its refusal names no way forward. Evidence: card 14 `TestReadRaddleStore` "unreadable `store.json` (error naming it)".

I made no further tool calls.
