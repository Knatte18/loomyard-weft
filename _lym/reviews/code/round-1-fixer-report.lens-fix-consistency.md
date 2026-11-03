```yaml
lens: fix-consistency
outcome: reported
```

Task: check the `_lym/landing/summary.md` rewrite against its surroundings for consistency (lens fix-consistency).

no findings

The changed sentence agrees with the code and with the rest of the summary:
- **Code match:** `Dirs.Records` is documented as "the absolute path of the records checkout … no layer reads it". The summary's "no config layer reads" matches. `ConfigDirs` sets `Records` from `RecordsWorktree(l)`, so "the weft checkout" is the same checkout.
- **Caller match:** `internal/hibernatecli/cli.go:101` and `:120` still call `ConfigDirs` and `hibernateconfig.LoadConfig`, as the new sentence says.
- **Summary siblings:** "The merge adds no new behaviour" (line 4), the four-file list and the Verification section don't contradict the new exception. The kept stencil claim is unchanged and still stands beside the `PATTERN` sentence.
- **Twins:** the summary has no second place that repeats the "none of the beam's code that main's commits read changed" claim.

Tool calls:
1. `grep -n -B6 -A8 "Records" internal/configengine/layers.go`: I read it to confirm the `Dirs.Records` doc comment says no layer reads the field. My context only held the review's description of it.
