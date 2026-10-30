```yaml
lens: fix-claims
outcome: reported
```

Task: check every claim the fix added or rewrote against the code at HEAD (fix-claims lens).

No findings.

The claims I checked:
- Card 14 says `internal/mergeresolve/spec.go` fills its template with `stencil.Fill`. That holds: the file imports `internal/stencil` and calls `stencil.Fill` at line 95.
- Card 14 says landingshed needs a new `internal/stencil` allowlist entry. That holds: no production file in landingshed imports `internal/stencil`, and `landingshedAllowedImports` doesn't list it.
- Card 14 says a plain append keeps the row off `internal/languageconfig`. That holds: landingshed doesn't import it and its allowlist doesn't list it, so a plain append is needed for the gate to pass, unlike `mergeresolve`, which uses `languageconfig.AppendSkills`.
- Card 11 quotes two recipe sentences. Both match the files: `contracts/recipes/darn-recipe.yaml:27` exactly, and `contracts/recipes/loom-recipe.yaml:36-37` across its comment line break.
- Card 13 says step 6 makes no `Status` read in two cases: a first entry, and a kept `ClosedForContinue` record. That agrees with card 11's step 6 as written. In both of its `ClosedForContinue` branches, `Status` is never read.
- Card 13 says `Call` reads the injected `Now`. That holds: card 13 adds `Now` to `PullRequestDeps`.
- The test row says step 6's read is the one poll of the first minute for a record seeded with `Continue`. That agrees with step 6, which reads `Status` whenever a non-`ClosedForContinue` record exists.

Tool calls:
- One grep of `internal/landingshed` for `languageconfig` and `internal/stencil` imports, in production files and the allowlist. My context had only grepped the allowlist file for `stencil` and nothing for `languageconfig`.
