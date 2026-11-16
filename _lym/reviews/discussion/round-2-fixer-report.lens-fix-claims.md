```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

**BLOCKING: D1's lasting copy gets swept, so "Bound: nothing removes those copies" is false**
- Changed text: D1, "It also writes a copy named by the run id into a `manifests` directory at the run-directory root, which a done run's cleanup leaves in place … Bound: nothing removes those copies." D11 now points at that copy, and so does acceptance criterion 3 ("the copy remains after a done run's cleanup").
- Evidence: `internal/shuttleengine/rundir.go:324-357`. `sweepOrphans` lists every directory at the run-directory root. A directory with no `run.json` counts as an orphan ("A directory whose run.json is missing or unreadable counts as an orphan"). Once it passes the age guard it gets `os.RemoveAll`.
- A `manifests` directory at that root has no `run.json`, so the sweep deletes it along with every copy inside. The run-end stamp survives only because it is a file, and the sweep skips anything that isn't a directory (`if !entry.IsDir() { continue }`). The comparison "as it leaves the run-end stamp beside it" therefore doesn't hold for a directory.
- This leaves F4's premise unresolved: the record still disappears after a done run.

**MEDIUM: D12 counts lookup-only cells toward `lookup_run`, but Claude counts single-lookup messages**
- Changed text: D12's first bullet, "a lookup-only cell, every nested call of which is a lookup, counts toward `lookup_run` as a lookup-only message does". The last acceptance criterion's "`lookup_run` lookup-only `exec` cells" has the same problem.
- Evidence: `internal/shuttleengine/claudeengine/batchingnudge.go:37`: "LookupRun is the number of consecutive single-lookup messages". Line 470: "kindLookup is a message that holds exactly one lookup call and nothing else".
- Claude never counts a message that holds several lookups. Under the fixed text, a Codex cell that batches five lookups would count toward the breach, which nudges the very batching the nudge asks for. The rule should be "a cell whose only nested call is one lookup", which matches what Claude does.

**LOW: "as Claude's subagents do" is not checked against the code**
- Changed text: D12, "so each child has its own count and cap, as Claude's subagents do".
- `batchingnudge.go:166` keys the count by `agentID` ("the nudge count of the chain agentID names"), which supports per-chain counts. I made no further call to confirm the cap is per chain as well. I'm not raising it as a defect.

Other changed claims held: the `functions` namespace tool list, the per-nested-call hook firing and payload fields from the spike, `parsed_cmd` marking `cat` and `sed -n` as `read`, `EngineSet.DefaultName`, and `codex-file`'s re-render list.

Tool calls:
- `grep` of `rundir.go`: to find the sweep and the run-end stamp's location, which my context showed only in part.
- `grep` of `batchingnudge.go`: to read Claude's `lookup_run`/`edit_run` semantics, which my context never read.
- `sed` of `rundir.go:318-365` plus a `grep` of `kindLookup`: to read the sweep body and the single-lookup definition, which my context lacked.
