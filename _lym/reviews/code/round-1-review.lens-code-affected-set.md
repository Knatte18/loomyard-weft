```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: I traced everything outside the changed lines that reads, calls or describes what the diff changes. Nothing outside the diff is left wrong, so this lens has no findings.

**Per change, the affected set and whether the diff covers it:**

- **`Desktop.Add` hub launcher name** (`desktoplauncher.go:53`, now `lym: &lt;shortname&gt;`)
  - Callers: `internal/dobbyengine/launcher.go:64` in `addLauncher`, which is reached on every dobby start, and the test helpers `internal/dobbyengine/launcher_test.go:134` and `internal/machinecli/start_test.go:620`. The signature `Add(lymPath, shortname string)` is unchanged, and none of these reads the file's text: a grep for `Name=` in those tests found nothing. Covered.
  - Package tests: the rows of `TestAdd_WritesLaunchersRunningMachineStart` are updated on every form, and the `wantAll` rows still expect `lym: start every hub`. Covered.
  - Docs: `internal/desktoplauncher/doc.go:19` is updated in the diff. `docs/overview.md:347`, `docs/overview.md:368` and `docs/overview.md:912` describe the launchers without naming their displayed name, so they are not stale. Covered.
  - A grep of the repository, `_lym` included, found no remaining "start the hub" in any stencil, spec, fixture or sandbox suite. The only hits are unrelated Slack-relay wording at `internal/slackcli/cli.go:527` and `internal/fabriccli/anchor.go:61`. The only hits under `_bolt` are the board's own title of this task, at `_bolt/README.md:133` and `_bolt/project-root.md:15`, which describe the change itself. Covered.
- **`content` parameter rename `purpose` → `title`, and its doc comment**
  - `content` is unexported; its only callers are the two lines of `Desktop.Add` (`desktoplauncher.go:53`, `desktoplauncher.go:56`), both in the diff. Covered.
- **`cmdFile` parameter rename and doc comment**
  - `cmdFile` is unexported; its only callers are the two branches of `content` (`desktoplauncher.go:164`, `desktoplauncher.go:166`), both in the diff. Covered.
- **The repository's own rules**
  - No `PATTERN.md` entry or `pattern/` file covers launcher naming. No new path or name constant is needed: the file names `hubPrefix` and `allHubsName` are unchanged, and the displayed titles are text, not paths. Covered.
- **Declared PATTERN changes**
  - The directive declares none, so nothing is raised.
- **Commit messages and departures**
  - Commit `5c2dd5549` carries no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer, and none is needed. The test rows change only their expected names; no limit is loosened. Covered.

Tool calls:
1. A grep of `_lym`, `docs`, `pattern`, `PATTERN.md`, README and `_bolt` for the old name and launcher prose. My context had only searched Go and repository files outside `_lym`.
2. A read of `docs/overview.md:905-925` and of the `_bolt` README and project-root hits, to check whether the launcher passage names the displayed name.
