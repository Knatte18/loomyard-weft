---
round: 1
focus:
  - "internal/landingshed/fixer.go `FixerTable`: confirm `languageconfig.AppendSkills(fixerSkills, in.LanguageSkills)` returns a fresh slice, so the package-level `fixerSkills` is never aliased or mutated across calls, and that its dedup behavior matches how `mergeresolve` and the other code-writing seats combine base and language skills."
  - "Scope against the record's Fix and Same-family sections: the diff changes `internal/samplerengine/spec.go`, a seat the record does not name. Check whether the record's 'check every other seat that spawns a writer' sentence licenses adding skills there, and whether the sampler really writes and commits code."
  - "Seat coverage: the record asks that every seat spawning a writer be checked and named in `_lym/landing/summary.md`. Grep the repository for other `Skills:` assignments and skill-list vars, and judge whether any code-writing seat the summary lists as unchanged or omits still lacks the base skills."
  - "Tests: `TestFixerTable` must show the seat's skills start with the base skills in both modes, as the record asks. Check `mergeresolve_test.go` and the landingshed integration tests for coverage of `scribe:testing`, and for assertions that are duplicated or stale."
  - "Doc and comment accuracy: `internal/mergeresolve/doc.go`, the `conflictSkills`, `fixerSkills` and sampler `skills` comments, and the `FixerInputs.LanguageSkills` comment must match the code. Check the repository's docs and stencils for other places that list these seats' skills and were not updated."
---

Seed pass only; the focus entries name where to look and leave verdicts to the reviewer.
