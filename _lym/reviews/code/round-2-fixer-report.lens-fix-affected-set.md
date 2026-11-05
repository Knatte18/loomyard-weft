```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The F1 set is the places that word the durable-vs-ephemeral entry. I found two: the overview line `PATTERN.md:13` and the background file `pattern/PATTERN-durable-vs-ephemeral-state.md`.

- **Overview line:** it now carries "rendered files" and `.heddle`, as the review's fix key asked. It also says "untracked", where the line said "never-tracked" before.
- **Background file:** it still uses "never-tracked" (lines 3, 6, 7) and "rendered instruction files … `.heddle` temporary directory" (line 16). The review calls that wording the full version, and the overview line does not contradict it. "Untracked" in the overview is a looser form of the same word and changes no meaning.
- **Other sites:** nothing else under `pattern/` or `internal/pattern/` repeats the changed phrases. `tools/` script "variable block" appears only in the background file (line 22), not in the overview, so shortening it to "variables" in the overview leaves no stale twin.

Calls made:
1. `grep -rn "never-tracked\|rendered instruction files\|rendered files" PATTERN.md pattern/ internal/pattern/`: it listed every site that words the changed phrases, which my context lacked.
