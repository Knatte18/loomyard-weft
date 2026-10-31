## Interview

A darn run holds no interview.

## Rejected alternatives

- Measuring on OSL-1046: the board entry fixes Neve, since the count is machine-independent.
- A third gate run per tier to observe each count: the cap is two runs, so the rest is derived from the package graph.
- Running `--upto 2` on the default diff scope: the branch has no diff, so no tier would run.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, self-pick): what "whole suite" means. Picked `--upto 2 --all`, the non-manual tiers over all targets.
- Q2 (resolved, self-pick): what `--upto 2` scopes to. Picked an explicit small package set, because the diff is empty.
