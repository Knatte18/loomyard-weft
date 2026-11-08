```yaml
lens: fix-affected-set
outcome: reported
```

Findings (fix-affected-set):

- **LOW** — `contracts/stencils/webster/webster-body-implementer.md:131`: the sentence "A failure the card did not cause is never yours to fix, and it is left to the plan-level `webster` gate." has no exception. The fixer added the exception at lines 107, 132 and 175, but left this twin unchanged. It sits directly above line 132 and states the same rule unconditionally. The new text at line 107 does say "the one exception to the rules below", which covers it. A reader would still find "never yours to fix" contradicting the last batch's added packages.
  - Evidence: the grep for "did not cause" shows three sites in the stencil (131, 132, 175). The fix touched two of them.
  - Fix: add the same "except …" clause at line 131, or word it "never yours to fix, bar the exception above".

- No findings for the other set-premised findings, from what my context shows:
  - F1/F2: the only `renderBatchGate` callers are `beginbatch.go:562`, `recoverbatch.go:286` and `cardgate_test.go:154`, and each passes `isLastBatch`. The begin and recover paths are now both pinned.
  - F4/F5: their sets are the branches the review listed, and the new tests cover each. The two directory-error branches in `ModuleMainPackages` were not in the fix key.

Unchecked: none.

Tool calls:
1. `grep -rn "did not cause"` over the md and go files: my context held the stencil lines but not other sites that repeat the rule.
