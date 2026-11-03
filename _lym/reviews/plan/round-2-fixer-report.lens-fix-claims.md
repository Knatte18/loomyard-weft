```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every sentence the fix added or rewrote against the code at HEAD.

no findings

The claims I checked all hold:
- **Card 1, the `storeVerbs` upsert bullet:**
  - The `upsert` `Use` token sits in `storeVerbs` and ends in `on_land?`.
  - Its `Long` lists every accepted payload key, `beam`, `front` and `on_land` among them.
  - `upsertPayload` is built from `boardengine.UpsertPayloadKeys()` (`internal/boardcli/cli.go:459`), so `TestPayloadUse_MatchesDeclaredKeys` fails while the token and that list disagree.
- **Card 5:** `getCmd` (`internal/boardcli/cli.go:701`) is inside `storeVerbs`, which starts at line 475. `rename` (`internal/boardcli/cli.go:201`) is inside `Command`, so keeping `Command` on card 5's Edit list is right.
- **Cards 8 and 9, the comment on the dropped error:**
  - `preRun` (`internal/loomcli/decision.go:860`) calls `resolveReviewTarget` before it builds `s.deps`.
  - `resolveReviewTarget` refuses `recipeUnknown`, which `serviceOfRecipe` returns exactly when `IsLoom` errors.
  - `IsLoom` and card 6's accessors parse through the same `parseEmbeddedRecipe`, so any recipe the accessors cannot parse is already refused upstream. That includes the empty name, which reads as tapestry.
- **Card 7, the new sentence:** after an accepted-round resume the run's row is back inside Discussion-Review. Card 9's not-past refusal therefore covers it, which agrees with the new decision record entry.

Further tool calls:
- `internal/boardcli/cli.go:475-556` and the `upsertPayload` greps: my context held the `Use` line but not the `Long` text or the source of `upsertPayload`.
- `internal/loomcli/decision.go:836-930` and `internal/loomcli/arm.go:140-180`: my context showed only grep lines for `preRun`, and none of `serviceOfRecipe`.
- The `loomrecipe` parse-path greps (`IsLoom`, `parseEmbeddedRecipe`, `HasEngine`) and `internal/boardcli/cli.go:455-464`: I needed to confirm that the accessors and `IsLoom` share one parse.
