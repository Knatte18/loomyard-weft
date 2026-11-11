No findings.

I checked F1 to F8 against the code, and none of the proposals breaks anything:

- **F8:** `UpdateJSON` returns the mutate error unwrapped and writes nothing for it (`internal/state/state.go:173-176`). The new `!errors.Is(err, errParentAskUnchanged)` guard in `AnswerParentAsk` therefore passes every refusal that returns `errParentAskUnchanged`, which writes nothing, straight through. It returns the wrapped write error only for a real failure. On the past-deadline path, where mutate returns nil, a failed write is now reported as a write error instead of the refusal. A report-write failure still arrives as a plain error with no refusal set. The `refusal != nil` return after the guard cannot see a failed write.
- **F7:** the marker sits directly above the func line in the same form as the other `//testtiming:keep` markers in the repo, for example `contracts/stencils/registry_test.go:117`.
- **F1 to F6:** the rows distinguish what they claim to.
  - The F3 admitted-caller row uses `ly:orch`, which differs from the ask's strand `ly:orch-asked`, so it takes the parent-match branch and not the strand-match branch.
  - The F3 unresolved-parent row uses a caller that differs from the ask's strand, so it reaches the `parentErr` branch.
  - The F4 and F5 rows reach distinct branches.
  - F2's P2 expectation (no severity, no class) matches what the code writes, because both fields are `omitempty`.

Calls made:
1. Read `state.UpdateJSON` and grepped for `testtiming:keep` markers: my context held neither the mutate-error behaviour nor the marker form the repo uses.
