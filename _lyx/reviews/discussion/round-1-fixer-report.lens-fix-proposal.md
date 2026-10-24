```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (fix-proposal lens).

- **MEDIUM**: the F1 acceptance criterion can't be tested on Windows and gives an unreliable result on Linux.
  - Changed text: Acceptance criteria, the `tier1` bullet: "checks … that the call returns when its build and test processes exit while a reused MSBuild node or compiler server is still running".
  - Case 1 (Windows): Decision 7's Windows bullet says nodes and the compiler server "end with each build or tier run", because the tied child's kill-on-close job ends them. So on Windows no node is still running when the call returns, and the condition the criterion asserts never holds.
  - Case 2 (Linux): whether a node or VBCSCompiler is alive when the call returns depends on the SDK. It depends on whether the fixture build is multi-node and on whether the shared compiler started. To assert it, the test must probe for processes it didn't start and wait on a real-time bound. scribe:testing ("No wall-clock … dependence"; "A test that passes only most of the time is a bug") rules that out.
  - The criterion also tests the SDK's stdio handling, which Decision 7 now records with sources, not a behaviour lyx implements. Decision 7 adds no lyx-side wait bound, so the criterion pins nothing lyx owns.
  - A sound version would limit the assertion to Linux, or drop the assertion that a node is still alive and keep only "the call returns once its own processes exit".

- No finding on F2, F3, F4, F5, F6, F7, F9, F10, F11 (restore part), F13, F14 or F15. For each I built a case and the proposal held:
  - F5: the closure already reaches a derived class through the base name its file spells.
  - F13: a tier reverting to default `carry` gets back its original fingerprint, whose entries are legitimate passes.
  - F9: `--ignore-exit-code 8` on a whole run matches VSTest's own default of exit 0 on zero tests.

Tool calls: none.
