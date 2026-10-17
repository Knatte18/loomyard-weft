# Support log — lens fan and twill fixer for Webster, Merge and Darn review

## Interview

A darn run holds no interview: the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- Keep `FanList` in `loomengine` and give `darnengine` its own copy: rejected, it duplicates the grammar, its shape error and the fan validation, and two copies drift.
- Have `darnengine` import `loomengine` for `FanList`: rejected, it pulls the loom engine into darn's config module for one type.
- One shared `code_fan` key for all three code perches: rejected, the board entry asks for per-perch keys, and a hub may want a fan on one perch only.
- Name the darn keys `darn_fan` and `darn_fix_fan`: rejected, `darn.yaml` already scopes them and its budget keys are `review_*`.
- Relax the fork audit to refuse every git command: rejected, the board entry asks the stencils to match the audit, which refuses only mutating git.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, self-pick): Where does the shared fan grammar live? `internal/burlerengine`, which owns fan resolution and imports neither engine.
- Q2 (resolved, self-pick): Which model keys does a Merge or Darn row's engine refusal name? Webster's, the keys its models come from.
- Q3 (resolved, self-pick): Which lenses do the code fans carry? `review-code`: `code-correctness`, `code-affected-set`, `code-tests`, `code-pattern-fit`; `fix-code`: `fix-code-correctness`, `fix-consistency`, `fix-proposal`, `fix-affected-set`.
- Q4 (resolved, self-pick): Are the new keys closed to the local layer? Yes, as the existing fan keys are.
