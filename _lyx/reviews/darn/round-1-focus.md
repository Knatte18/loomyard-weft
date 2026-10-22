---
round: 1
focus:
  - "internal/yamlengine/set.go: does `SetValues`/`SetSparse` handle `P.<name>.<rest>` for names the template lacks and for names it holds, and are refusals (unknown rest, shape mismatch, scalar written over a mapping) correct and covered?"
  - "internal/configengine/layers.go `ValidateLocal`/`ValidateMachine`: does judging a path under a per-name map against the template entry holding the rest of the path leave other per-name maps (lenses and others) unchanged?"
  - "internal/yamlengine/reconcile.go and fill.go: do the commented-empty-mapping fixes (`key: {} # comment`, block style on append) emit valid YAML in every caller path, and does each fix have a regression row?"
  - "internal/batcher/config.go: does the retired-key refusal name the right layer file now that `profiles` merges by name, and does the board entry's scope (#833, #834) match what the diff changes, with nothing beyond it?"
  - "internal/configsync/configsync.go: does narrowing the former batcher template value to `profiles.cautious.budget: 450000` preserve the former-values contract, and does the registry fingerprint change match the claim in `_lyx/landing/summary.md` about hub reconcile?"
  - "Docs landing in the same diff: `doc.go` of batcher, configengine and yamlengine against the new behavior, and any PATTERN.md invariant the change introduces."
---

Seed focus drawn from the diff stat and the change description; the summary's claims are to be checked against the diff.
