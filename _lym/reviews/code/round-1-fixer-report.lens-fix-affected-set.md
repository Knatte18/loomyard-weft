```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the finding searches for F1 and F2, the two findings whose premise is a set of sites.

**Findings**

- **LOW**, `internal/hubgeom/slackgeom.go:46` and `internal/hubgeom/dobbygeom.go:34`: both still call `os.Executable()`. The result goes into `slackengine.Geometry` and `dobbyengine.Geometry`, whose consumers may name the binary in a spawned or pinned command. The amended `PATTERN-child-processes` exemption lists only the loomcli drive wiring and battencli wire, so these two sites match neither the rule nor its exemption text. The same goes for `internal/tendercli/deps.go:98` and `internal/adoptcli/deploy.go:73`, though those look like reads or comparisons of the binary itself, which the rule allows. The F1 review did not list any of these sites. I did not trace what the hubgeom geometries do with the path.
- **LOW**, `pattern/PATTERN-child-processes.md`, the exemption sentence: the review's own `fix` asked to name the resolve-once exemptions, and the amended text does. It is silent on the hubgeom geometry sites above, so the rule and the code still do not fully match.

**Checked, no gap**

- **F1:** all five named lym spawn sites and the vscode stamp now go through `selfexec.LymBinary` (or the `LymBinary` seam). No `os.Executable` call is left in `creelcli`, `gatecli`, `reedcli/remove.go`, `reedcli/window.go`, `boardengine` or `ideengine`. The `reedcli/watchdog.go:656` call is the own-binary stat the review excluded.
- **F2:** every other `executablePath` use in `reedengine` (`window.go:108`, `spawn.go:126`, `bindings.go`) goes through a `compose*` function that calls `ResolveLymBinary`. The button prompt was the only raw one, and it is fixed.

**Not checked**

- F3, F4 and F5 have no set premise, so I did not rerun a search for them.

**Tool calls**
1. Grepped every non-test `os.Executable` and `syncExecutable` use in the repo, to find sites the review did not name.
2. Grepped `executablePath` uses in `reedengine`, to confirm no other raw call.
