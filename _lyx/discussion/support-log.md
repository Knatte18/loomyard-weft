# Support log — test-suite-derived-guards

## Interview

The session ran autonomously: no operator answered questions, and every pick below is an auto-pick recorded in the ledger.

1. **Task and context.**
   Read the board entry, #323, #326, #327, part 1's decision record (weft archive tag `archive/test-suite-change-amplification/5c93b11126fc`) and the dependent board tasks `pair-lifecycle`, `verify-publish-hardening`, `review-finding-class` and `pattern-init`.
   The three tier-1 tasks after this one depend on it; none duplicates its scope.
   `pair-lifecycle` owns production socket and session cleanup, `verify-publish-hardening` owns card verify scope, and `pattern-init` owns the `CONSTRAINTS.md` move and the test-file vocabulary scan.
2. **Operator decision on #326 §2.**
   Mid-session the orchestrator (`ly:orch`) relayed the operator's approval: derive the CLI-tree and registry tests from the real registries, delete the per-CLI copies, keep `registration_test.go` as the mount oracle, and treat changing the CLI/Cobra Invariant's enforcement as in scope.
   Section 2 is designed in full (D5–D8).
3. **Harness shape.**
   The leaf and seam guards are the same import-allowlist scan.
   Weighed one table in `cmd/lyx` against short per-package callers; picked per-package callers (D3), since a card's package-scoped verify must still catch a package's own import violation, and `verify-publish-hardening` builds on that.
4. **Path guards.**
   Production has no registry of path constructors (108 production references to the two `lyxdirs` constants).
   Picked one table of constructor closures with a class, with expectations computed from the rule (D7), over a production registry or an AST-discovered constructor set.
5. **Plan fixtures vs the Planparser Sole-Parser Invariant.**
   A test-only plan writer conflicts with "sole parser and writer".
   Picked restating the invariant to sanction exactly one test-side writer, `plankit`, bounded to tests (D10).
6. **Socket leak mechanism.**
   Production reed addresses tmux only by `-L <key>`, and `OwnsTmuxEnv` compares the basename only, so `TMUX_TMPDIR` relocates every socket a test causes, including those of child processes and reed panes.
   Picked per-package isolation through `TestMain` plus a sweep of the kit's own directory (D17), enforced by a scan keyed on "has a tagged test file" (D18).
   Packages run as concurrent processes, so the sweep never touches another directory.
7. **Testkit exemption for `tmuxkit`.**
   Killing a server needs `tmux kill-server`, so `tmuxkit` needs `os/exec`; picked a second bounded exemption beside `lyxbin` (D19) and a Tier Purity rule that keeps its spawning exports out of untagged files (D18).
8. **validate-plan duplication.**
   `loomcli` and `webstercli` render findings into different envelope shapes, so #327 §6's condition ("act only if the two envelopes can share a mapper") is not met; no action (D14).
9. **Wide positional functions.**
   `RenderIntegrationPrompt` is likely retired by `verify-publish-hardening`; picked local test helpers and no production signature change (D16).
10. **Invariant audit.**
    Walked every `CONSTRAINTS.md` section bearing on the touched tests and recorded change or keep for each (D20).
    The directive asks for each invariant that forces duplicated tests or hand-kept copies to be restated, moved to one shared scan or dropped; per-package scan copies and registry mirrors were enforcement shape, not rule text, so most change only in enforcement.
    Kept in text: Cwd Resolution and Durable-vs-Ephemeral (D7 changes only the test's shape), Shed Recipe Registry (its tests already range `Names()`), Agent Name (no hand-kept mirror found), Stencil Ownership (D11 routes test seeding through its owner), Sandbox Suite Coverage (already derived from `newRoot()`; its exclusion list carries reasons) and Shed Verb-Set (D14's bound moves any `loomcli` case `shedverbs` lacks before deleting it).
11. **Churn evidence behind the derived tests.**
    About twenty scans each rebuilt the same root, walk, allowlist and floor scaffolding, and the leaf guard files differed only in package name and allowlist (D1).
    The `go env GOMOD` exec forced an allowlist entry per untagged guard file (D4).
    `helptree_test.go` was edited in 29 of 32 commits that touched it, each for a one-word list addition (D5); `configreg`'s list took the same one-word edit in three commits (D6).
    gofmt re-aligned about twenty lines per added allowlist key (D8).
    One recipe row edited three tables in `internal/loomrecipe` (D9).
    `format: 5` appeared in ten test files outside `planparser` (D10).
    `websterengine/template_test.go` alone carried over a hundred phrase checks, each re-reading the stencil (D12).

## Rejected alternatives

- **One import-guard table in `cmd/lyx` for every leaf and seam package.**
  Fewer files, but `go test ./internal/<pkg>` would stop catching that package's import violation, and the `CONSTRAINTS.md` sections naming per-package enforcers would go stale.
- **Keeping `go env GOMOD` root resolution in the harness.**
  It is a spawn in untagged files and is what forced the per-file Tier Purity allowlist entries.
- **A production registry of path constructors, or `lyxdirs` helpers through which every path is built.**
  Either is a production refactor across many packages for a test-only gain; the D7 table gets the derived expectation without it.
- **AST discovery of path constructors with a completeness check.**
  It would force an edit (a classification) for every new constructor, which is the amplification this task removes.
- **Keeping the exact-verb-set tests.**
  They catch only an accidentally deleted verb, and the operator accepted that trade-off with #326 §2.
- **A test-only writer outside `testkit`, or leaving the plan helpers as they are.**
  Fifteen helpers already write the format; one kit under the Testkit importer rule is the smallest sanctioned surface.
- **An options struct for `RenderIntegrationPrompt`.**
  The function is likely retired by `verify-publish-hardening`.
- **Sharing a findings mapper between `loomcli` and `webstercli` `validate-plan`.**
  The envelopes differ, and unifying them changes observable CLI output.
- **Per-test `tmux -S <path>` sockets under `t.TempDir()`.**
  Production reed uses `-L`, so tests that go through reed, the binary or panes could not use `-S`; `t.TempDir()` paths can also exceed the 108-byte socket limit.
- **A before/after comparison of `/tmp/tmux-<uid>/` in each `TestMain` that fails on a new `lyx-*` socket.**
  Packages run concurrently and the operator's live hubs create `lyx-*` sockets there too, so the comparison is racy; isolation removes the leak instead of detecting it.
- **Failing the package when the sweep finds a live server.**
  It would make every test responsible for its own cleanup again, and the sweep already guarantees no leftover.
- **Keying the D18 rule on "starts tmux".**
  No static shape captures it: the `lyx` binary and reed start servers indirectly.
- **Sweeping other processes' stale `lyx`-prefixed temp directories.**
  A concurrent package's live directory looks the same, so it could kill another package's servers mid-run.

## Review rounds

_No rounds yet._

## Question ledger

| # | Question | Status | Answer |
|---|---|---|---|
| Q1 | Does #326 §2 go ahead, and may the CLI/Cobra Invariant's enforcement change? | resolved (operator, relayed by `ly:orch`) | Yes, as #326 §2 proposes; `registration_test.go` kept. |
| Q2 | One `cmd/lyx` import-guard table, or per-package callers? | resolved (auto-pick) | Per-package callers of a `scankit` assertion (D3): keeps package-scoped verify catching violations. |
| Q3 | Where does the harness live? | resolved (auto-pick) | `internal/testkit/scankit`, stdlib only (D1, D19), per the board body. |
| Q4 | Which scans convert? | resolved (auto-pick) | Every scan that walks source or resolves the root (D2), with stale detection and a floor added. |
| Q5 | How are path guards derived with no production registry? | resolved (auto-pick) | One table of constructor closures plus class; the rule computes the expectation (D7). |
| Q6 | Keep or delete the exact-verb-set tests? | resolved (auto-pick) | Delete (D5), with the bound stated. |
| Q7 | How does `plankit` square with the Planparser Sole-Parser Invariant? | resolved (auto-pick) | Restate the invariant to sanction `plankit` as the one test-side writer (D10). |
| Q8 | Is `validate-plan`'s duplicated testing collapsed? | resolved (auto-pick) | No: different envelope shapes, #327 §6's condition unmet (D14). |
| Q9 | Options struct or local helper for wide functions? | resolved (auto-pick) | Local helper per test file; no production signature change (D16). |
| Q10 | How are tmux sockets kept from leaking? | resolved (auto-pick) | `TMUX_TMPDIR` isolation per test package in `TestMain` plus a sweep of the kit's own directory (D17). |
| Q11 | How is the isolation enforced? | resolved (auto-pick) | Tmux Test Isolation Invariant, scan keyed on packages with tagged test files (D18). |
| Q12 | May a second kit import `os/exec`? | resolved (auto-pick) | Yes, `tmuxkit`, bounded to `tmux` against its own sockets (D19). |
| Q13 | Unset `TMUX`/`TMUX_PANE` in tests? | resolved (auto-pick) | Yes, so no test reaches the caller's tmux server (D17). |
| Q14 | Which invariants change under the directive? | resolved (auto-pick) | D20's list; the rest audited and unchanged in text. |
| Q15 | Is cleaning the existing dead sockets on the operator's machine in scope? | resolved (auto-pick) | No: a one-off `rm`, not code. |
