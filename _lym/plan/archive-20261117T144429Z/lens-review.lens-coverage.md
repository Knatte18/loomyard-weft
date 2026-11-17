```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked each decision, acceptance criterion and Intent branch against the planned test rows.

**Findings**

1. **MEDIUM. D1: the manifest is read from Prepare's inputs, not from the launch.**
   - The record says each engine reads its manifest "back from the launch its `Prepare` produced: the launch command, the files it wrote and the settings it passed".
   - Card 3 (`claudeManifest(in manifestInputs)`) and card 4 (`codexManifest(in manifestInputs)`) build items from "the values the launch line was built from", such as `toolHooks`, the item list and the user setup.
   - So a delivery dropped inside `buildLaunchCmd`, `hookSettings` or `appendPromptText` leaves its item in place, and no test fails.
   - The first acceptance criterion requires that "dropping any one Codex delivery from `Prepare` fails the test". No row reaches a delivery dropped after the inputs are built.
   - Evidence: `_lym/plan/03-claude-manifest.md` Intent line "manifestInputs, which carry the values those artifacts were written from"; `_lym/plan/04-codex-manifest.md` Intent line 22.

2. **MEDIUM. D1: most compared Claude items are never pinned, so the parity test can pass vacuously.**
   - `TestSeats` and `TestProducers` only check that Codex holds every item Claude holds. If Claude's manifest leaves out a channel, both pass.
   - Card 3's `TestPrepare_WritesTheManifest` rows assert only these items: guards, the question deny, skill loads, notices, the fork deny, the batching nudge being off, one passage, and settings.
   - No row asserts any `instruction-source` item (the native global or above-root CLAUDE.md, root-to-anchor CLAUDE.md files, `heddle-file/<slug>`). None asserts `skill-listing`, `plugin-context`, `prompt-engine`, the `hand-written` key or `subagent-passage`.
   - Card 4's rows likewise skip `instruction-source`, `standing-passage`, `skill-listing`, `batching-nudge`, `prompt-engine` and `subagent-passage`. They also skip the `claudeFileIsUnread` delivered-by-pointer branch.
   - Evidence: the card 3 Tests paragraph and the card 4 Tests paragraph.

3. **MEDIUM. D1 operand: the repo root Claude keys instruction sources by is unstated.**
   - Card 4 keys the CLAUDE.md files above the root by absolute path, and the checkout-path files relative to `found.root`. Its `WorkDir` is `filepath.EvalSymlinks(c.dirs.Anchor)` (`codexengine.go:191`).
   - Card 3 keys "each CLAUDE.md above the repository root" and "from the checkout root to the anchor", but never names where Claude gets the repository root or whether the anchor is symlink-resolved.
   - The Claude engine reads no checkout today; `Claude.Prepare` holds no `readCheckout`.
   - If the anchor or temp dir is unresolved, the absolute-path keys differ between engines, and items show as missing falsely, or match falsely.

4. **MEDIUM. Fidelity, assumption "D1: no exemption table": the plan adds an engine-keyed item the record does not license.**
   - The record keys an item to one engine only for single-engine heddle targets and settings.
   - Shared Decision "the Claude fork deny is keyed to Claude" (`00-overview.md`) and card 3 key the `lym webster` fork deny to `claude`.
   - The parity tests also skip every spec that requires forks. Card 6 says such a seat "is skipped"; card 7 says "as in card 6". That drops all compared items of those seats, such as Merriam, whose `websterengine#Run` literal sets `ForkSubagents` and `Requires`.
   - No row asserts the skip branch.

5. **MEDIUM. D11 operand: the plan writes to `codexengine` where the record names `codex-engine-trial`.**
   - The record's D11 and its review criterion name `codex-engine-trial`. Card 10 writes to `codexengine` on the board's evidence, with no `Added after Discussion` entry.
   - The prompt's Step 1 route says a change of decided scope is planned only once the parent records it.
   - Card 10 also tells `codex-writer-bench` that the "single-session bounds no longer hold". That contradicts its own `## The one run` text ("Exactly one Codex session"), which the card leaves unchanged ("nothing else of its body changes").
   - Card 10 drops D11's "its replay route is redesigned before it reruns".

6. **MEDIUM. D10 operand: the verb's seats are built from stand-in inputs, not the current worktree's.**
   - D10 says the verb prints "the manifest each registered engine gives it in the current worktree".
   - Card 6's catalog builds every table with "a placeholder slug and paths under `env.Dir`, zero configs".
   - So `in.LanguageSkills` (Darn's writer, the fixer, the fix reviewer) and the configs' role and model settings are empty. The skill-load and skill-listing items the verb reports for a seat are not the ones a real launch of that seat gets.
   - This seat is exactly the Darn-Write writer that `codex-writer-bench` checks.
   - Evidence: card 6 Intent "stand-in inputs"; card 8 Intent "resolves the seat catalog with `Seats` over the hub's stencils directory".

7. **MEDIUM. D10 criterion: the verb's success and failure exits are never run.**
   - `TestManifestCmd_RefusesAMissingOrUnknownSeat` covers only a missing argument and an unknown seat.
   - No row runs `manifestCmd` to its `output.Ok` path, which prints both manifests, or to its `output.Err` non-zero exit naming `<channel> <key>`. The criterion is "prints both engines' manifests for a seat, exits non-zero naming a missing item".
   - `TestReport` checks only `Missing` at the library level.

8. **MEDIUM. Edit-directive channel: the not-delivered branch is unreached.**
   - Card 6's `PromptItems` adds `edit-directive` "delivered when the prompt holds the engine's copy".
   - No row covers a prompt that lacks the copy. `TestCompare` uses hand-built manifests, and `TestSeats` only sees prompts that contain the copy.
   - So a `PromptItems` that always adds the item passes every test.

9. **LOW. D1 scan: the orch skip has no row.**
   - Card 7's `TestLaunchSpecLiterals_AreListed` rows cover the `internal/shuttlecli` and `internal/testkit` skips.
   - No row covers `internal/orchcli` or `internal/orchengine`, so dropping that skip fails only the real-module row through `orchcli/start.go:217`, and only by accident.

10. **LOW. Unreached branches in card 2.**
    - `copyManifest`'s write failure, "logged at Warn and never fails the launch", has no row.
    - Its handling of an unreadable record, as opposed to an absent one, is unstated and untested.

11. **LOW. Unreached branches in card 5.**
    - The nil-seam fallbacks have no row: `RunHook` falling back to `claudeengine.RunPluginHook`, and a nil probe keeping Codex's own.
    - No row checks that `New` passes zero seams; only the given-seams row exists.

12. **LOW. Card 1: two `Passage.Engine` branches have no row.**
    - An entry aimed at both a shared target and a single-engine target has no row; the rule is "only through a target aimed at that engine".
    - `StandingPassages` returning the zero `Standing` on a reader error has no row.

13. **LOW. Spec-level denies.**
    - No manifest test row installs `DenyPublish` or `DenyPython`, so the switch-dependent guard rows are listed unchecked on both engines.

14. **LOW. Card 6's `PrepareEach` has two error branches with no row.**
    - An absent record "is an error naming the engine", but no row reaches it.
    - A `Prepare` error is reached only through card 8's `TestReport`.

15. **LOW. D10 bound wording.**
    - `UnrunHooks` returns an error, so Codex's `resolveUserSetup` warns for each hook with "way forward: fix the hook or its hooks file". The verb's output then tells the operator to fix hooks that are only unrun.
    - Evidence: `codexengine/usersetup.go`, the `resolveUserSetup` warn loop.

I made no further tool calls.
