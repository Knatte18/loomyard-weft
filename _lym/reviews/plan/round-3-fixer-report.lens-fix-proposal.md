```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F2 and F3, which the fixer carried out as proposed.

**MEDIUM: F3, card 11's Recipes paragraph, rewrite of the `loom-recipe.yaml` header sentence**
- **Summary:** the proposal changes only the reason PR-Review carries no `on_stuck`. It never takes PR-Review out of the sentence's "outside the escalate-to-a-human set" frame, so a literal rewrite still makes a false claim.
- **Evidence:** in `contracts/recipes/loom-recipe.yaml:19-23`, the escalate-to-a-human set is the rows that carry no `on_stuck` and whose "Stuck halts the whole run just as a gate's does". Once every Stuck of PR-Review is a Halt, PR-Review belongs in that set.
- **Breakage:** the card says only "each keeps the never-Stuck claim for Friction-Reflect and says PR-Review carries no `on_stuck` because its Stuck is always a Halt". An implementer can keep "PR-Review and Friction-Reflect are outside the escalate-to-a-human set: …" and change only the clause after the colon. The header would then still say PR-Review is outside the set.
- **Better change:** the card should say PR-Review leaves that sentence and joins the set of no-`on_stuck` rows whose Stuck halts the run, with Friction-Reflect alone kept outside.

**LOW: F1, card 14 line 52, "built with a plain append so the row never imports `internal/languageconfig`"**
- **Summary:** a plain append drops the de-duplication that the sibling `internal/mergeresolve/spec.go:107` gets from `languageconfig.AppendSkills`.
- **Evidence:** `internal/languageconfig/config.go:146-161` shows that `AppendSkills` skips a language skill already in `own` and repeats none. `Deps.LanguageSkills` comes from operator config, and if it holds `scribe:prose` or `scribe:conversation`, a plain append loads that skill twice.
- **Better change:** have `ensureSession` call `languageconfig.AppendSkills` and add `internal/languageconfig` to `landingshedAllowedImports`, the same way the card now adds `internal/stencil`.

F2: no findings. I tested three cases:
- **Step 6 reads merged or closed:** the first wake uses that status, and the session starts and then stops. Card 14 already allowed this.
- **Kept or reopened `ClosedForContinue` record:** `entryStatus` is nil and the first wake polls.
- **Continue seeded before `Call`:** the starting copy already carries `Continue`, so no transition poll fires.

None of them breaks the new rule or the new test rows.

Tool calls:
1. `internal/languageconfig/config.go` `AppendSkills`, and `LanguageSkills` use in `internal/landingshed`: needed to see what a plain append gives up. My context had only the import at `spec.go:107`.
2. `internal/languageconfig/config.go:120-146` and a search for where `scribe:prose`/`scribe:conversation` are configured: needed the `AppendSkills` doc comment, and whether language skills can repeat the session's own skills.
3. `contracts/recipes/loom-recipe.yaml:15-37`: needed what "escalate-to-a-human set" means in that header. My context held only lines 33-38.
