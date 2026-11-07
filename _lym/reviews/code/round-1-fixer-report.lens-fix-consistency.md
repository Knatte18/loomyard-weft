```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set (`template_test.go`) for agreement with its surroundings.

**Findings**

- **LOW** — `packRepositories` doc comment, `internal/hubforge/template.go:257`: it says the repositories hold "no loose object and no loose ref".
  - The test's `HEAD` exclusion, with its comment "A symbolic ref such as refs/remotes/origin/HEAD stays loose after pack-refs", shows that claim is not literally true.
  - The test and the production comment now disagree on what "no loose ref" means. The test narrows it to branch refs other than symrefs.
  - The sentence at `template.go:257` could say "no loose branch ref", to match the test.
  - `doc.go:33-36` makes no loose-ref claim, so it is consistent.

**No other disagreements**
- The test's doc comment ("no loose object or a loose branch ref") matches what the test asserts.
- The file header "the packing of a built template" still fits.
- The test name `…HoldsNoLooseObject` is now narrower than what it asserts. That is cosmetic and not worth a finding.

**Tool calls**
1. `grep` for pack-refs, packed refs and loose in `doc.go` and `template.go`. My context did not hold those sibling files, and the lens needs their claims about refs.
