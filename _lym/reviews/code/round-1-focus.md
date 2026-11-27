---
round: 1
focus:
  - "Landing coverage: trace every route by which work reaches main or a parent (parent-align, finalize, adopt, beam, PR and darn paths, scope all with inert_paths, green-record and inert carries) and settle which of them runs the `parent-align finalize` checks over the final tree."
  - "Held and marked tests are matched by bare name (internal/gatecli/only.go, tiers.go, withoutHeld): settle what happens when one name exists in several packages, tiers or modules with different keys, and when a tier's module is not the one scanned."
  - "Green record with `held` (internal/gatecli/green.go): settle whether coverage, bases, inert carries and entry replacement stay right for listed runs, guard-step runs, marked-scope runs and runs whose args carry no skip, in both directions: no pass over a skipped test, and no lost carry the diff promises, such as Finalize over Parent-Align."
  - "Marked scope end to end: settle that `lym gate test --gate upkeep` runs exactly the marked tests under any flag combination, lint, empty or uncovered-tier case, and that a run that executes none of them cannot read as a clean upkeep."
  - "Computed and --base guard step (scope.go skippedInside, impactset targets.go InsideGuards): settle whether a guard test of an unlisted tier now runs in every case, including whole shares, tiers with no base, several base groups, several modules and watched tests."
  - "Where each check runs: check the markers against every whole-repository guard and slow check the board entry names or covers, in cmd/lym and in other packages, including unmarked or `//lym:watch` tests, and settle whether each has the place the entry asks for."
  - "Board-tidy upkeep step (dobbyengine, beamcli/dobbydeps.go, the beam stencils): settle whether the agent can finish the upkeep call and file notes within its timeout, permissions and the stencil's other rules, how main-branch detection holds, and what a failed or unrendered call leaves behind."
  - "Retiring `create-caller-later`: search for leftover references, symbols and docs that served only that check, and weigh the deleted cmd/lym test file against the rubric's loosened-checks rule."
  - "Scan cost and failure paths of withOnlyTests: settle behaviour on every --gate call when go list or the marker parse fails, when the call has -C or explicit targets, and when a marker names a key a hub's gate config lacks."
  - "Docs and claims: check gate, config-template, PATTERN and spec text against the code, and the change description's claims and `## Merge danger` door and blast radius against what a revert or an older binary does with the new records and config values."
---

Seed focus for round 1.
The decision record holds only the board entry under `## Goal`, with no amendment, so that entry is the measuring stick.
