---
round: 1
focus:
  - "Parallel safety in internal/webstercli and internal/loomcli tests: for each test or subtest that newly calls t.Parallel, does it, or any code it reaches, touch process-global state (environment, working directory, PATH, process-wide logger, package variables), including env reads in production code the diff leaves unchanged?"
  - "Parent and subtest lifetimes where cases became parallel subtests (verify_integration_test.go, verify_parity_integration_test.go, reset_test.go, the commitstatus and discussion-record tests): are environment settings, hub copies, loop variables and cleanups still in force and isolated when each subtest body runs?"
  - "Values passed in instead of read from the environment (cli.go, wiring.go, sync.go, ownership.go, batchpause.go, reset.go and their callers): does every path that reaches a sync, a reset, the orphaned-Master guard or the master-strand lookup run after wiring has set syncOptions and strandName, in both hub and standalone wiring, with unchanged production behaviour?"
  - "Tests that set environment variables before: where a test now sets struct fields or drops the setting, does it still run under the same skip switches and strand name, and is the wiring-time read of those variables covered by any test?"
  - "Assertions kept: compare every removed or rewritten assertion, row and test against its replacement, including the folded TestWireLanding table and the TestVerifyVerb subtests, and check that no check was dropped or loosened."
  - "Moves to the pair-carrying hub template in internal/loomcli: does each moved test start from the same state its own AddPair gave it (slug, branches, remotes, written files, config), so that its assertions still check what its name and doc comment say?"
  - "Scope against the amended entry: how do the loomcli asks (share one built hub copy between read-only tests, fold rows of the same shape) and the webstercli asks (pass environment or working-directory values in, including the tests the change description lists as still serial) compare with what the diff does?"
  - "Change description _lym/landing/summary.md: do its claims about which tests stay serial and why, what was left unchanged, the before/after figures with load averages, the merge-danger door and blast radius, and its wording about tooling hold against the diff and the rubric?"
  - "Changed and new doc comments and test comments: do they follow the repository's conventions, read as intact text, and still describe the tests they sit on after the renames, folds and parallel changes?"
---
Seed focus for round 1, drawn from the amended entry, the commit messages, the diff stat, the change description and the production-code hunks of the range.
The test diffs have not been read.
