---
round: 1
focus:
  - "internal/loomcli/projectrecord_integration_test.go `beamHubRegistration.Build`: does the step order and content match the former `beamHub` steps exactly, and does the uncommitted `board.json` and shortname record survive `hubforge.CopyRegistered`, so the six `beamHub` sites see the same board state as before?"
  - "internal/loomcli/smoke_helpers_test.go `badReedUpRegistration.Build`: does it equal the former `newBadReedUpFixture` steps (corrupted `mouse:` line, `SeedConfig` map, `AddPair`), and does a path or value baked into the seeded config or pair at build time (the template root) stay valid in each copy?"
  - "Acceptance criterion on registrations: do either build call `os.Setenv`, `os.MkdirTemp`, `git config --global`, keep `tb.Context()` past the build, call a package-global seam setter or once-only registration, or reach code whose result a once-only registration changes?"
  - "Acceptance criterion on seams: does any test using `beamHub` or `newBadReedUpFixture` install a process-global seam, and does it take its copies before installing it?"
  - "Each site of `newBadReedUpFixture`: does the caller's `seed` still run on its own copy after `lymcwd.Resolve`, with the same assertions as before?"
  - "Do the tests that call `Add` still add their own pairs, and does any caller of `beamHub` rely on a slug or pair the registration's `Base`/`Build` no longer provides, or provide twice?"
  - "Does `hubforge.Registration` get a unique `Key` for each template, and is `Registration` use (`Key`, `Base`, `Build`, `CopyRegistered`) consistent with the API as `internal/hubforge` defines it?"
  - "Summary claims in `_lym/landing/summary.md` (site counts of six and four, and the `newWiredPairFixture` two-site exclusion): does the diff and the surrounding code bear each out?"
---

Seed pass. The diff is two files in `internal/loomcli`, both moving a fixture's build steps into a `hubforge.Registration`.
