---
round: 1
focus:
  - "Gate reach: in internal/gatecli, internal/impactset and internal/loomshed/verifygate.go, does the moved TestGofmt_ModuleFilesAreFormatted run at tier0 on every task gate path (card, narrowed, impact-derived, whole), including a diff that does not select internal/testkit and a diff whose only mis-formatted file is a test or build-tagged file?"
  - "Walk coverage: in internal/testkit/scankit, does Walk with the All filter, called from internal/testkit, still reach every Go file under cmd, internal and tools (the answer key's set), and what does it skip? Does moving the caller's package change the root or the exclusions?"
  - "Repository rules: read PATTERN.md and pattern/PATTERN-test-speed.md and PATTERN-test-isolation against the move. Check marker placement (guard, watch, only), whether internal/testkit is an allowed home for a whole-repository guard with its imports, and whether any doc the rules name for this kind of change is missing from the diff."
  - "Stale references: does any file outside _lym still point at the old cmd/lym location of the guard, or describe formatting as checked only at landing (specs, stencils, pattern files, package docs, help text)?"
  - "Finding message: is the suggested `lym loom fmt <file>` command correct for every finding gofmtFinding can emit and every path the walk can report (module-relative path versus the caller's directory, files the verb refuses)?"
  - "Claims: check each statement in _lym/landing/summary.md and the commit message against the code, including the package-size and import claims, the extra directories covered, the unwatched-guard rule and the Merge danger door and blast radius."
---

The diff is small: a guard test moves package, loses its landing-only marker, and gains a fix hint.
The main risk is whether the guard runs where the answer key wants it, which the diff text alone does not show.
