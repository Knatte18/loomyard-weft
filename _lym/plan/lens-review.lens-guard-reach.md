```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: I traced whether each guard, refusal, branch and test row the plan adds can be reached, and whether a guarded site recurs unguarded elsewhere.

Findings:

1. MEDIUM — D3, D6 (card 9): what `lym loom status` prints is not covered; only the pane's checklist is.
   - The verb `lym loom status` is the generic shedverbs status: `internal/loomcli/arm.go:70` routes "status" to shedverbs, and `RenderStatusHuman` prints `last <Activity.Last>` at `internal/shedverbs/human.go:64-65`.
   - `Activity.Last` is composed by `internal/shedengine#composeActivity` (`internal/shedengine/activity.go:35`) as `<producer> → <outcome>`. A block row's Done therefore reads `Motif → done`, not entered, and an inlined row's line carries no block.
   - Card 9 changes only `renderChecklist` and its helpers, which draw the pane. Card 1 leaves `composeActivity` untouched.
   - The record's "`lym loom status` shows a block row's Done entry as entered" and "shows an inlined row's block beside its name" therefore go unguarded on the verb's own output.

2. LOW — card 8: the new refusal is reachable, but it now comes before the existing, clearer one.
   - Seed-Child calls `writeRunSettings` right after `ReadBoardType` (`internal/battenshed/seamchild.go:66-74`), before the `childRecipeSupported` check behind `WriteSeed` (`internal/battencli/wire.go:531-538`).
   - A board task naming an unknown recipe now halts on `loomrecipe: no embedded recipe named %q`, wrapped as `ErrInvalidRunSettings`. That refusal names no way forward.
   - Before this change the same task reached `unsupportedChildRecipeRefusal`, which lists the admitted recipes.

3. LOW — card 2: the empty-name branch of `plan:internal/shedrecipe#LookupBlockGate` can never be reached.
   - Card 3's `checkRowShape` refuses a block row missing `block_gate`, and `Expand` keeps the value, so `Build` never passes an empty name.
   - The branch is harmless as long as it shares the unknown-name text. No test can reach it through `Build`.

4. LOW — card 5: the "block row is not expanded" guard covers `Build` only. The same unexpanded state passes unguarded through `RoutingOf`.
   - `RoutingOf` parses and delegates to `RoutingFrom` without expanding. A recipe holding a block row would project a block row whose `OnDone` is its exit and no inlined rows, with no refusal.
   - Today's only `RoutingOf` callers read the batten recipe (`internal/battenrecipe/battenrecipe.go:47`), so no shipped path hits it.

Tool calls:
- Read `childSeedParams`, the order of `Call` in `battenshed/seamchild.go`, the direct recipe parses outside loomrecipe, and the history writers in failedstop.go and rename.go. My context lacked the Seed-Child order and the child-params body.
- Grepped what renders the `lym loom status` verb. My context lacked whether the verb or only the pane draws the checklist.
