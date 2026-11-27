```yaml
lens: fix-affected-set
outcome: reported
```

I checked that the fix covers every member of each set-based finding. I left F3, F7, F12, F15, F17 and F16 unchecked; their premises are single sites or ran past my tool-call cap.

**Findings**

1. **LOW**, card 9 Intent: "`beamcli#hubLandDeps`' `PushBeam` refuses while the beam's own ledger key holds landings".
   - **Summary:** the F10 set of doc lines that say `PushBeam` and `lym beam pr` push the beam branch is only partly covered. Card 9 edits `hubLandDeps` and `internal/beamcli/doc.go`, but three comments describing the same behaviour stay stale and are not in card 9's Edit list.
   - **Evidence:** the stale comments are:
     - `internal/beamcli/land.go:50`: "PushBeam pushes the beam branch and returns the head it pushed", the `landDeps.PushBeam` field doc.
     - `internal/beamcli/pr.go:45`: "With none recorded, it pushes the beam branch and finds an open pull request from it".
     - `internal/beamcli/pr.go:152`: "prUpdate pushes the beam branch and sets …".

2. **LOW**, card 7 Intent: "`TestStartDetachedPush_RegistersTheChild`'s pair row expects owner `runproc.HubOwner` …".
   - **Summary:** the F11 set of spawn-owner assertions has a second member. `TestPushSpawnCommand` also carries `wantOwner` rows for pair requests (`PauseSlug: "task", TeardownSlug: "task"`). Card 7 lists that test under Edit, but its Intent only says the test "pins the pusher command lines" and never says these rows' owner changes to the hub owner and the push role.
   - **Evidence:** `internal/fabricengine/spawn_test.go:61` declares `wantOwner`, `:65`, `:71` and `:77` are pair rows, and `:104` asserts `owner != tt.wantOwner`.

**Sets confirmed covered**
- **F1:** the production callers of `PushPairAnchored` are `weft_verbs.go:314`, `DetachedPush` (`teardownlock.go:98`) and `pairteardown/handover.go:95`. All of them meet the hold now that it lives in `PushPairAnchored`.
- **F9:** every user of `ArchiveParentMerge` is removed by card 12: `landingshed` deps and finalize, the `envkit` key, `landingdeps.go`, `handover.go`, `doc.go` and the integration test.
- **F13:** `pairteardown/doc.go:168-174` is covered by card 9 and card 19, which now list it under Edit.

**Tool calls**
1. grep of the production callers of `PushPairAnchored` and `DetachedPush`, for F1's caller set; my context had only `weft_verbs.go`.
2. grep of every file naming `ArchiveParentMerge`, for F9's set; my context had only the Go hits, no `.md` hits.
3. grep of the `wantOwner` rows and the `PushRun` callers, for the F11 and F10 sets.
4. read of the test names in `spawn_test.go` and of `PushTip` in `codetipsettle.go`, to name the second owner test and check whether a beam push is reached.
5. grep of the `pairteardown` and `beamcli` doc lines and the `PushBeam` users, for the F13 and F10 doc sets.
