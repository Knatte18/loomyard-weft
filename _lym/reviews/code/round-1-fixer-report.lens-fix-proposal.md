```yaml
lens: fix-proposal
outcome: reported
```

**Task:** challenge the proposals the fixer carried out for F1, F2, F3 and F4.

**Findings**

1. **MEDIUM — `internal/adoptcli/doc.go`, the sentence "The steps run in order: start, each tier's start and end, then end."**
   - **Summary:** The sentence gives the wrong steps, copied from the review's `fix` text. It describes the check's record steps, not the adopt's.
   - **Evidence:** The adopt's steps are `hold`, `pairs`, `takeover`, `catchup`, `deploy`, `workspace`, `check` and `beam` (base doc.go:37-80). `internal/adoptcli/check.go:147` says "The steps are its start, each tier's start and end, and its end" about `runCheck`. The `lym adopt --help` text at `cli.go:47` ("the package doc holds the steps") points at the adopt's steps. The new sentence therefore contradicts the following sentence, "Each step after the first pulls Bolt…", and leaves the help text's claim unmet. The fixer applied the reviewer's wording without checking it against the base doc.
   - **Better change:** Name the adopt's eight steps in order, and put "start, each tier's start and end, end" only under `# The check`.

2. **MEDIUM — `internal/yamlengine/doc.go`, the new overview opening "no operation replaces a value the file holds".**
   - **Summary:** The claim is false for the set operations.
   - **Evidence:** `SetValues`, `SetSparse` and `SetEntries` write a value over the file's value, and `# Open maps` itself says they "replace it whole". `FollowTemplate` also takes the new template value for units the file left at the old one. The sentence "The file's values always win over the template's" holds only for `Reconcile`, `FillMissing` and `MissingKeys`.

3. **LOW — `internal/yamlengine/doc.go`, the sentence "The loaded value is unchanged by the names Reconcile drops, so a later template edit to that name reaches the file."**
   - **Summary:** The sentence is garbled and carries a rationale the doc comments no longer back.
   - **Evidence:** It reads as if dropping names leaves the loaded value unchanged, which is the opposite of the reason: the dropped name resolves from the template at load, so the value is the same. It is also a kept fragment with no lead-in beyond the refusal sentence above it, so it reads as part of the refusal's description.

4. **LOW — `internal/batcher/doc.go`, the cut of the read-set and `PeakContext` term list.**
   - **Summary:** The cut removed every statement of the `StartBase` zero value and of Webster's role in computing it.
   - **Evidence:** The cut paragraph included "The start base (StartBase) is what the orchestrating session loads… the zero value adds nothing". The fixer did not confirm that `StartBase`'s own doc comment states this. Context shows only `PeakContext`'s doc (`estimate.go:207-214`), which mentions neither the zero value nor Webster. If the `StartBase` doc lacks it, the contract is lost.

5. **NIT — `_lym/landing/summary.md` (F4).** The "162" → "161" change is correct. Neither it nor the other findings' fixes were checked against `git ls-tree -r --name-only HEAD | grep -c 'doc\.go$'`. The file is under `_lym`, so the fixer correctly left it uncommitted. The report should say that.

**Tool calls**
- One `git show 3d3b4812:internal/adoptcli/doc.go | grep …` and one `grep` over `internal/adoptcli/*.go`: my context held only the elided base steps and lacked the source of "start, each tier's start and end".
- One `git show … | sed -n 37,80p` and `sed -n 144,150p internal/adoptcli/check.go`: to read the base step list and the check's steps, which my context lacked.
