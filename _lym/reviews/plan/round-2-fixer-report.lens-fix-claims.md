```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checking the factual claims in card 8's rewritten Intent sentences against the code at HEAD.

no findings

Each claim holds at HEAD:
- **The kept name passes `prepareRunSettings` with settings alone when the task names no profile.** `profiles.Default` returns `p.Defaults[recipe]`, which is empty for an unknown name (`internal/runprofile/runprofile.go:96`). That leads to `configreg.PrepareRunSettings(settings.Settings)` (`internal/battencli/runsettings.go:28-33`).
- **No step between run settings and the seed write refuses first.** The call order is `writeRunSettings`, then `ChildDriver`, then `WriteSeed` (`internal/battenshed/seamchild.go:73,84,92`). `childDriverKind` ignores the recipe (`internal/battencli/arm.go:183`), so `ChildDriver` does not refuse an unknown name.
- **`WriteSeed` refuses the name through `shedrun.ValidateRecipe`, wrapped in `battenshed.ErrUnknownRecipe`, and names the available recipes.** The wrap is at `wire.go:1392-1393`, and the "available recipes" text is at `shedrun/seed.go:96`.
- **A task that names a profile is refused by `profiles.Settings` because the profile holds no block for that name.** The refusal is at `runprofile.go:88-90`. A named profile that does not exist is refused earlier by `Lookup`, but that refusal also comes from inside `Settings`, so the sentence still holds.

Tool calls:
1. Read `prepareRunSettings` and grepped the runprofile refusals: I needed the run-settings branch logic, which my context did not hold.
2. Read the `seedChildProducer.Call` and `writeRunSettings` order in `seamchild.go`: I needed to confirm no step between them refuses an unknown recipe.
3. Read the `ChildDriver` wiring and `Profiles.Settings`: I needed to know whether `ChildDriver` refuses an unknown recipe and what text `Settings` refuses with.
4. Read `childDriverKind` and the `Settings` body: same reason as call 3, to finish that check.
