---
round: 1
focus:
  - "internal/battencli/beamguard_integration_test.go `beamTemplate` build: do its three `AddPairWith` steps match the former per-test steps exactly (options, order, slugs), and does the build avoid every call the decision record's Review acceptance criterion forbids (`os.Setenv`, `os.MkdirTemp`, global git config, keeping the TB's `Context()` past the build, package-global seam setters, once-only registrations)?"
  - "Every test in internal/battencli that calls `beamHub` (including the one near line 237 that does not install the replay): does each take its copy before any process-global seam is installed, per the acceptance criterion on seam ordering?"
  - "Does the `Registration` meet D7's other conditions: the `Key` is unique across registrations, the `Base` shape equals the old `CopyHub` shape, and no step belongs in the test instead (a path outside the hub, a board HEAD left ahead of or behind its upstream tip, a push into a bare from outside the board)?"
  - "Scope against the Goal: does the diff touch only `beamHub` and leave `notices_integration_test.go` untouched, and do the commit message's claims (steps moved as they were, copy taken before the replay in two tests) hold in the diff?"
---

Seed focus for a one-file, test-only diff.
