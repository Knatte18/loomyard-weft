# Support log — fork-git-guard-read-only-tag

## Interview

A darn run holds no interview; the decision record is written from the board entry and a reading of `internal/burlerengine/cluster.go`.

## Rejected alternatives

- **Drop `tag` from the mutating set.**
  Rejected: `git tag v1` creates a ref, and the operator decision keeps creating and deleting forms refused.
- **Treat any non-option word after `git tag` as a tag name to create.**
  Rejected: in list mode such words are patterns, so `git tag -l 'v1.*'` would stay a false positive.
- **Also classify `git branch` and `git stash` by their arguments.**
  Rejected: the operator decision scopes the change to `git tag`; recorded as an open risk.
- **Rewrite the twill fixer's way forward for a clean working tree.**
  Rejected: once the false positive is gone, a refused command did write state, so the way forward stays accurate.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Which read-only `git tag` forms pass? Self-pick: every flag git documents as implying list mode, plus bare `git tag`.
- Q: Does a tag-writing flag beside a list-mode flag pass? Self-pick: no, the command stays mutating.
- Q: Are combined short-flag clusters such as `-ln` parsed? Self-pick: no, they stay mutating, which errs on the refusing side.
- Q: Are `branch`, `stash`, `remote` and `config` changed? Self-pick: no, per the operator decision.
