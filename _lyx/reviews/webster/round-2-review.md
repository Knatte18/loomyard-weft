---
verdict: APPROVED
---

No findings.
The range is `git merge-base main HEAD..HEAD`: `_lyx/loom/rework/` does not exist, and `product.parent` in `_lyx/shed/webster-audit-gaps/status.json` is `main`.
`lyx gate test` over `./internal/fabricengine`, `./internal/websterengine` and `./internal/webstercli` passes.

## Focus answers

**Git config flag rows.**
The row `git config --get --file=f k` has three argument words, so it passes the two-or-three-word bound in `readOnlyGitSegment`.
Its first word `--get` is accepted, and it is rejected only by the `-`-prefixed check over `args[1:]` on `--file=f`.
Removing that check turns the row red.
The other config rows (`--unset a`, `user.name x`) are rejected at the first-word check, which is the behavior they pin.
The rows for `show --output=x`, `diff --output=x`, `remote set-url o u` and `branch -m a b` each reach their form's flag or listing check.

**Pathless class list.**
In `ClassifyViolation`, the only class that is correctness with an empty `Path` is `fabric-reference`; the other pathless classes (`nested-agent`, `named-spawn`) are policy and never reach `Uncheckable`.
`pathlessViolationClasses` covers all seven class constants, so the over-inclusion is harmless.
The three evidence readers skip a pathless entry: `splitContractPaths` and `contractWritesFor` in `contractevidence.go`, and `findingtext.go`; `auditledger.go` matches the entry by class.
`TestCheckFork` and `TestCheckParent` round-trip every emitted violation through `pathlessEntryClass`, and their cases emit all seven classes, so a class removed from or misspelled in the list turns a row red.
A new class constant that no test case emits would not be caught, which is acceptable for a guard tied to the audit's own emitters.
