```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked that every refusal, next step, row and test the fix introduced can be reached and leads somewhere useful.

**[MEDIUM] D1's `manifests` copy is swept away, and the way forward in D11 leads nowhere**
- **Changed text:** D1 says the copy goes "into a `manifests` directory at the run-directory root, which a done run's cleanup leaves in place … Bound: nothing removes those copies." D11 opens its report "from the copy D1 keeps in the run-directory root's `manifests` directory." The D1 acceptance criterion says "the copy remains after a done run's cleanup".
- **Summary:** The orphan sweep deletes any directory at the run-directory root that has no `run.json`. That includes `manifests`.
- **Evidence:**
  - `internal/shuttleengine/rundir.go:319`: "A directory whose run.json is missing or unreadable counts as an orphan".
  - `rundir.go:353-355`: `if err != nil || !found || !strandGUIDs[rs.StrandGUID] { os.RemoveAll(runDir) }`. This runs once the directory's mtime is older than the age guard.
  - So "nothing removes those copies" is false, and a bench that reads its record after a quiet spell finds nothing. That is F4's premise coming back.
  - `findRunByStrand` (`rundir.go:268-290`) also counts `manifests` as an unreadable run directory. Every not-found lookup would then return the "run.json is damaged … inspect those directories" error instead of `ErrNoRunRecord`, sending an operator to inspect a directory that is not a run.
  - The criterion "the copy remains after a done run's cleanup" passes a test while the copy still disappears at the next sweep.
  - To fix it, D1 needs one of two changes: say that the sweep and the strand lookup skip `manifests`, or keep the copies outside the run-directory root.

**[LOW] D9's dynamic import triggers have no mechanism in the static target table**
- **Changed text:** D9 says "The `codex-file` target's re-render triggers also hold every file the repository CLAUDE.md imports, as the expansion resolves them".
- **Summary:** `ReRender` is a fixed per-target list in a static table, matched against event paths. D9 does not say where the per-worktree import set lives or how the matcher gets it, so the step as written cannot be carried out on the existing seam.
- **Evidence:** `internal/heddleengine/target.go:65` gives `{Name: codexFileTarget, … ReRender: []string{"AGENTS.md", "CLAUDE.md", "AGENTS.override.md"}}`. `internal/heddleengine/seam.go:56` matches `target.ReRender` against `ev.Paths`. Nothing there is computed per worktree.

I found nothing wrong in the other introduced items: the D4 default-engine call, D10's size-check bound (D5's refusal still names its way forward), the D12 deny and nudge criteria, and the Q14 pointer to the existing `Added after Discussion` entry.

Calls made:
- A grep of `ReRender` in `internal/heddleengine` and of the root functions in `rundir.go`. My context lacked how re-render triggers are matched and where the run root is swept.
- A read of `rundir.go:255-364`. My context lacked how `sweepOrphans` and `findRunByStrand` treat a directory without `run.json`, which the new `manifests` directory is.
