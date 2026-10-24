---
round: 1
focus:
  - "internal/gatecli/scope.go card branch tag scan: does it select tier N exactly when a card-package .go file matches under the single tag tierN and not under no tags, and does it skip a tier a listed tier is or covers?"
  - "internal/gatecli/tiers.go vet planning: does planTierRuns stay disk-free, and is the vet run placed after the listed tiers and before the guard step, only for --card under the go scoper (not none, computed, --base, --all, explicit targets)?"
  - "internal/gatecli/run.go: does a vet run hold a gate slot, use the tier's modules, env, parallel and timeout_min, write a record and summary line with scope form vet, join the green record under its own fingerprint, and name `vet tierN` in the closing line without stopping later runs?"
  - "Tests in scopes_integration_test.go and tiers_test.go: do they cover the acceptance criteria (a tier1 file using a tier0 helper fails with vet tier1; a clean package passes; no vet for listed, covered or no-added-file tiers) without duplicating existing coverage?"
  - "doc.go and docs/overview.md: do they describe the vet step accurately against the code, and is the change description's claim borne out by the diff?"
---

Seed focus from the board entry and decision record; the range is the gatecli change only.
