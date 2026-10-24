# Support log — webstercli standalone tests fail when an empty /tmp/.git exists

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- Seed a `.git` marker in every standalone test target: fixes the webstercli rows but not `cliwire`'s `NoRepositoryAboveIsReturnedUnchanged` row, which by design has no marker, and every future standalone test would have to remember it.
- Set the ceiling in `gitkit.HermeticGitEnv`: packages that spawn no git, such as `internal/cliwire`, never call it, yet walk the filesystem the same way.
- A lyx-specific ceiling variable: git already defines one with the same meaning, and git processes the tests spawn honor it too.

## Review rounds

_No rounds yet._

## Question ledger

- Should the fix hunt down the writer of `/tmp/.git`? Self-picked no: the origin is unknown, and the ceiling makes the tests immune to any stray marker above the run directory.
- Should the start directory be checked when it is itself a ceiling? Self-picked yes, matching git, which checks its starting directory before climbing.
