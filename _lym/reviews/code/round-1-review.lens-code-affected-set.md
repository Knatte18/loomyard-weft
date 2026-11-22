```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I traced everything outside the changed lines that reads the new `hardenLiveArgs` output.

**Per change, the affected set found:**

1. **`hardenLiveArgs` now adds `./` to package arguments that `build.IsLocalImport` rejects.** Everything that reads those arguments is covered:
   - `hardenLiveRunner` (`internal/loomcli/hardenwiring.go:298`) is the only production caller. It passes the args straight to the `lym` binary, so it needs no change.
   - `lym gate test` handles a `./internal/x` target. The gate itself builds `"./"+dir` package arguments (`internal/gatecli/tierrun.go:803`) and resolves relative run directories under the root (`tierrun.go:809-813`).
   - `NewLiveGate`, `copiesRun`, `packagesOutsideStrayCover` and the `Started == 0` check (`internal/hardenshed/live.go:131-195`) all read `deps.Packages`, the raw config entries, never the prefixed arguments. They are unaffected.
   - `CheckCampaign` (`internal/hardenshed/campaign.go:55,66`) joins each entry to the worktree root, so bare and `./` entries are both valid in config, as the record says.
   - The live-gate package doc (`internal/hardenshed/doc.go:41`) still holds: "the config's packages as targets".
   - The `Config.Packages` doc (`internal/hardenshed/config.go:31`) still holds: "worktree-relative directories".
   - The `harden-live` key in `internal/gateslot/template.yaml:44`, `pattern/PATTERN-test-speed.md:58` and `docs/overview.md` don't describe how package arguments are written, so none goes stale.
   - No stencil or spec describes the live call's package arguments; a grep over `.md`, `.yaml` and `.tmpl` found none.
   - The only test that asserts the args is `TestHardenLiveArgs` (`internal/loomcli/hardenwiring_test.go:41-60`), and the diff updates it.

2. **The `hardenLiveArgs` doc comment changed.** It has no readers outside its own symbol.

**Rubric checks:**
- **Repository rules:** no PATTERN entry is affected. The change adds only a stdlib `go/build` import to a `cli` package, not a leaf package.
- **Declared PATTERN changes:** the directive declares none, so nothing to check.
- **Commit messages:** the `Root-Cause:` trailer on d95d266ff names the defect the diff removes, entries appended verbatim. The second commit only re-breaks the comment. Neither carries a `Pattern-Departure:` or `Loosened-Check:` trailer, and neither needs one.
- **Stencil wording:** no stencil changed.

**Findings:** none.

**Tool calls:**
1. Grepped `.md`, `.yaml`, `.tmpl` and `.go` files for `hardener.packages`, `harden-live` and `hardenLiveArgs`, plus the doc and test call sites. My context had no list of the docs, stencils and tests that describe the live call.
2. Read the matching lines in `docs/overview.md`, `template.yaml`, `PATTERN-test-speed.md` and `hardenshed/doc.go`, and checked for stencils. This was to confirm whether those texts describe the format of the package arguments.
