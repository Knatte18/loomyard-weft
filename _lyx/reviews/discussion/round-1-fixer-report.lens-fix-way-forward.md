```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checking that each refusal, next step, row and test the fix added can be carried out and leads to progress.

Ways forward the fix introduced:
1. Decision 3: a non-empty seed with no class and no opted-in tier runs each listed tier whole, with the reason on stderr. This is reachable.
2. Decision 6: a tier with an empty test scope takes its slots and runs the build. This is reachable.
3. Decision 6: a failed build writes `build: true` to the landing failure record. This is reachable. The fixer prompt names the record's path (`internal/landingshed/fixerrun.go:175-178`), so the fixer reads the new key from the file.
4. Decision 7 Bound: `docs/dotnet-hub.md` tells hubs to leave the escape hatches unset. This is reachable.
5. Decision 8 Bound: the docs tell hubs to keep `carry: false` tiers out of darn's gate keys and out of every tier's `covers`. A hub can do both with `lyx config gate`, so this is reachable.
6. Decision 4: Microsoft.Testing.Platform hubs set `--ignore-exit-code 8` or `TESTINGPLATFORM_EXITCODE_IGNORE`. This is reachable.
7. The tier1 acceptance criterion: an offline restore with a skip, and a check for a lingering node. See finding 1.

Findings:

- MEDIUM. Acceptance criterion: "the call returns when its build and test processes exit while a reused MSBuild node or compiler server is still running".
  This check cannot be carried out on Windows.
  Decision 7's own Windows bullet says nodes and the compiler server "end with each build or tier run" in the kill-on-close job. So on Windows nothing is still running when the call returns, and the assertion either fails or passes without testing anything.
  To make it runnable, the criterion needs to limit the lingering-node check to Linux, or state what the test checks on Windows.

- LOW. Decision 8: "the sites the next Bound names stay that note's to avoid", together with the `docs/dotnet-hub.md` sentence that follows the Bound.
  The note is given no way to avoid one of the three sites the Bound names.
  The docs guidance covers darn's gate keys and `covers`, but not the verify site's verified-tree skip (`verifytree.Verify`, `internal/verifytree/verify.go:290`). That skip also applies to Merge-In and Finalize calls on a retry over an unchanged tree. Nothing tells the note or the hub how to keep a capture or compare tier clear of it, for example by running those tiers only through a direct `lyx gate test` call.

Tool calls:
- One grep over `internal/` for readers of `LandingFailure` and its `Tier` field. My context showed only the writer in `verifygate.go`, so I checked that the new `build: true` key reaches the landing fixer.
