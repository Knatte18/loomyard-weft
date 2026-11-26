```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each set finding's own search and check that the fix covers every member.

Unchecked: F1's write route through `configengine.Set` in `hardencli#commitSandboxConfig`. It is a write, not a load, and card 2 governs it. My one search of `Set`'s body returned nothing, so I could not confirm whether `Set` judges the whole file.

Findings:

1. MEDIUM. Card 2's new `pattern/PATTERN-config-layers.md` paragraph names line 7 and lines 77-78, but misses line 73.
   - **Summary:** The F5 set has a third member. Line 73 says "Only the operator removes a key the running template lacks, from a hub base or worktree file". That is still false once card 6's walk drops a moved old key, which the template no longer holds.
   - **Evidence:** `pattern/PATTERN-config-layers.md:73`. Card 6 says "`&lt;file&gt; dropped &lt;module&gt;.&lt;old&gt;, which was null`" and "`... since &lt;module&gt;.&lt;new&gt; is set`" for hub base and worktree files.

2. LOW. Card 9's `internal/hubreconcile/doc.go` paragraph rewrites only the opening and "Skips".
   - **Summary:** The F4 set has another stale member. `doc.go:21-22` says "The walk removes no key the running binary's template lacks ... The operator removes such a key with `lym config reconcile --apply` or `lym config &lt;module&gt; --unset &lt;key&gt;`". Card 6 makes the walk drop moved old keys, and neither card 6's Extends nor card 9's Edit text amends this.
   - **Evidence:** `internal/hubreconcile/doc.go:21-22`, card 6 Intent ("dropped ... since &lt;module&gt;.&lt;new&gt; is set").

3. LOW. Card 9's `doc.go` rewrite covers lines 3 and 33 but leaves line 18 saying "start verb".
   - **Summary:** `doc.go:18` says "The board's push goes to the detached `lym board sync`, so no start verb waits on a push". After card 9 every loading verb runs the walk, so the line now narrows the claim to start verbs.
   - **Evidence:** `internal/hubreconcile/doc.go:18`.

F1's set came back complete:
- **Loom and tapestry `LoadConfig` callers:** all sit in loomcli (covered by the pre-run or by `ArmAt` under shed), `beamcli#hubLandDeps`, `webstercli/wiring.go:119`, `shuttleparity/seats.go:178-182` and `configreg` `CheckLoad`, which only `configcli` calls and which is excluded.
- **`SettingLookup` callers:** loomcli, `battencli/wire.go` and `fabriccli/merge_verbs.go`, all covered.

F6's set is complete: `wantLocalDenied` holds the only loom and tapestry rows, since `wantLocalKeys` has none.

Tool calls:
- Grep of `loomengine.LoadConfig`/`LoadTapestryConfig`/spec callers: my context held only the reviewer's named sites, not the full set.
- Grep of `CheckLoad`/`SettingLookup`/`Lookup` callers: indirect load routes my context lacked.
- Function names around the `beamcli/deps.go:296` and `hardencli/cli.go:293` loads: to map them to card targets.
- Grep of `configengine.Set`'s body: to tell whether `hardencli`'s Set is a strict load. It returned nothing.
