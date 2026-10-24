---
round: 1
focus:
  - "internal/gatecli/green.go, run.go, tiers.go: does the fingerprint carry-over cover the new tier keys (filter, shared_build, carry, inert_paths, scoper dotnet), and does carry: false and inert-only history never let a stale green carry over a changed whole-gate result?"
  - "internal/gatecli/build.go and the shared build step in run.go: is the build run once per call inside the first opted-in tier's slots, and do its timeout line and failure record reach verifytree and the landing failure record with the build flag intact?"
  - "internal/dotnetimpact/lex.go, source.go, narrow.go, derive.go: does the C# lexer and type-name closure handle edge cases (strings, comments, partial and nested types, namespaces) without silently narrowing to too few test classes, and do the fallbacks widen to the whole solution on any doubt?"
  - "internal/gateslot/config.go: are the new gate.yaml keys validated with refusals for bad combinations, and does the inert matcher on Config behave identically for gatecli and dotnetimpact?"
  - "internal/gatecli/scope.go, dotnetscope.go: does the {filter} rendering and per-project invocation of the dotnet scoper match the decision record, including the --card form fed by internal/planparser/csharpseed.go?"
  - "Repository rules: do doc.go files, docs/overview.md, docs/dotnet-hub.md and pattern/PATTERN-gate-slot.md land with the code, and does every new behavior have test coverage without duplicating an existing test?"
---

Seeded from the plan's Card Index and the diff stat over the review range; no earlier round exists.
