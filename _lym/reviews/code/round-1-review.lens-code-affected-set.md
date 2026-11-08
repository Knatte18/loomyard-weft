```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. I looked for what the diff's markers and the `PATTERN-test-speed.md` bullets force outside the lines they touch. I found nothing the diff leaves wrong, so there are no findings.

| Change | Set found | Covered? |
|---|---|---|
| `//lym:watch` markers on 25 guard tests | The stencil guards depend on `contracts/stencils`, `contracts/orchrole`, `internal/stencil` and `internal/stencilstore`. `stencilstore` and `stencil` are flat packages. `contracts/stencils/stencils.go` imports only `stencilstore`, and `contracts/orchrole` is covered by its pattern. Both directive packages (`internal/parentdirective`, `internal/editdirective`) are flat and covered. | Covered |
| Directive packages used by the two directive tests | `internal/parentdirective/*.go` and `internal/editdirective/*.go`, which hold `MarkerName`. | Covered |
| Shared matcher between the two denied-recovery tests | `TestWebsterRetiredSteps_ScannedSurfaceHasNone` watches `cmd/lym/norecoverydeny_test.go`, which holds `deniedRecoveryTokenPattern`, `tokensMatch`, `hasAnyPrefix` and `deniedRecoveryHit`. `TestNoDeniedRecovery_ScannedSurfaceHasNone` depends on nothing in `websterretiredsteps_test.go`. | Covered |
| Shell guard dependency of `TestNoDeniedRecovery_ScannedSurfaceHasNone` | It reads `internal/shellguard/rows.go` through `Rows`, and the pattern watches `internal/shellguard/*.go`. `rows.go` also imports `gateslot` and `agentname`, but only for a regex constant (`lymEnvNamesPattern`), not for any `Forms` row the test compares. | Covered |
| Doc and PATTERN descriptions of watch markers | `internal/gatecli/doc.go` "Watched tests" describes the skip rule generally. The cross-compile example is an example only, so it stays true. The `PATTERN.md` index line already names `//lym:watch` per the background, so it needs no edit. The new bullets in `pattern/PATTERN-test-speed.md` sit in "Narrow tests carry `//lym:watch`". They agree with the "Repository-scanning tests carry `//lym:guard`" section, whose guard-plus-watch exception already existed. | Covered |
| Declared PATTERN changes | The directive lists none, and the `PATTERN-test-speed.md` edit is a background bullet change only. | Nothing to land |
| Commit message | One commit, with no `Pattern-Departure:` trailer, and none is needed. | Covered |

**Calls (3 of 6 used):**
1. A `find` plus import grep over `internal/stencil*`, `internal/parentdirective`, `internal/editdirective`, `contracts/orchrole` and `contracts/stencils/stencils.go` to confirm the packages are flat and have no unlisted imports. My context had not listed their files or imports.
2. A grep for `lym:watch` and `lym:guard` mentions in docs and `PATTERN.md`, to find prose that describes the markers. My context lacked those mentions.
3. A grep of the same `PATTERN.md` index line, which came back with call 2.
