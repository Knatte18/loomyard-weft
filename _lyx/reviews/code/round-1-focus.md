---
round: 1
focus:
  - "Green record (D4, cards 07-08): check that a carry needs a clean tree, an unchanged HEAD tree at write time, an exact tier fingerprint and exact args (bar the guard step's own `-run`), and that a carried tier takes no slot; settle whether a dirty or changed tree can ever read or write an entry."
  - "Diff base chain (D2, card 08): check that the base is the newest ancestor whole-green commit for the tier's current fingerprint, with empty args, falling back to the parent merge base and then to a whole run, and that a narrowed or `-run` entry never becomes a base."
  - "Scope values and config load (D1-D2, cards 01-02): check that `card` scope is refused off `card` and `batch` naming `lyx config gate`, and that `lyx config reconcile` maps `landing`, `final`, `round` and `guard_tier` per D1 and is idempotent."
  - "`--gate` flag and renders (D3, cards 02-06): check the refusals alongside `--tier`, `--upto`, `--all` and `--base`, the run record's gate key, each D12 site's key and `gate_key` override, and the verify fixers' comment-lint base rule."
  - "Darn tree scope (D14, card 06): check the three-way `tree_scope_verify` migration (held empty, held set, absent), and that tree-scoped sites render the row's key plus the touched trees' targets, rendering no call when none contribute."
  - "Tier tags and guard step (D5-D7, cards 11-16): check that guard-step grouping by file tier, the zero-test failure rule with its `-run`/`-skip`/`-list`/`-bench` exemption and platform-constrained files, and the one-tag scan's allowance for `|| integration` on guard-marked files only, all behave as written."
  - "Retag cards 23-31: sample for test files left untagged, double-tagged, or holding a test in an untagged or two-tier file; check split files whose helpers another tier uses."
  - "Shed entry gate (D8-D9, cards 17-19): check entered-row bookkeeping on resume, tackler or arbiter relaunch with and without a target, goto, route-back through skipped rows, `pause_before` and `pause_after`; that `skipped` is treated as passed-through by every history reader; chain-length stop; and that an evaluation error leaves the status non-running."
  - "Entry gate check form and imports (D9, card 19): check non-bool bare operands error with the setting's command, the build-time known-key check needs no task worktree, and `shedengine` imports nothing new."
  - "Consumers (D10, cards 20-21): check that batten's `--window` forcing works through the lookup, that `CommandDeps.Forced` and the switch read are gone, and that darn's Code-Review budget 0 settles the Bouncer approved with Describe skipped."
  - "Launch list and `lyx gate report` (D11, cards 09-10): check that cached and no-test-file packages are left out, launch gaps and elapsed come from package-level events, old records without a key or launch list group and render correctly."
  - "PATTERN and docs: check the declared PATTERN entries land with the declared text (the new and the shortened ones) and that docs (`doc.go`, `docs/overview.md`, the D12 table in `internal/gateslot`) land in the same diff."
  - "Test economy: new tests duplicating existing coverage, and behavior the diff adds with no test; the `tools/tokencount` fixtures split deserves a check that no test was lost."
---

Round 1 spans a large retag diff.
The behavioral cards (01-22) carry the review risk; the retag cards (23-31) are mechanical and need sampling only.
