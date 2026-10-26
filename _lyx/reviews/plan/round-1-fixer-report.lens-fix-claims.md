```yaml
lens: fix-claims
outcome: reported
```

fix-claims: checking the claims the fix added or rewrote against the code at HEAD.

**LOW: card 18, the new draft-skip bound overstates which callers can start the run**
- **Changed text:** card 18's "What it lets through" paragraph: "so only `lyx beam create` or a hand pass starts a beam project's run".
- **Problem:** the claim leaves out adopt's `ResumeRun`, which the same card lists as a `StartFirstPass` caller.
- **Evidence:** the card says "Its callers pass empty options: adopt's `ResumeRun` and `lyx beam land`'s `StartLanding`". Card 4's `movedRuns` adds "each adopted beam's own pawl run", and that run is then resumed through `deps.ResumeRun`.
- **Reach:** a draft beam whose pawl run record is not done can therefore be passed by adopt too. Name it beside `lyx beam create`, or narrow the claim to a fresh start.

These claims check out:
- **Card 3:** `TestFabricReplay` has two owner rows, at replay_test.go:335/337 and 352.
- **Card 4:** `TestDropLocal` has two anchor-change rows, droplocal_integration_test.go:112 and 119. `refuseDropLocalAnchorChange` takes only `(l, slug, weftPath, tracking)`.
- **Card 7:** the parent in `FailedDobbySpawnWarnsAndKeepsThePass` is `hub-repo`, the prime.
- **Card 9:** cli_test.go:92 sits in `TestPrimeSegments` (from line 56), package `cleanupcli`.
- **Card 15/17:** `dobbyengine.Live(geom Geometry) (bool, error)` exists at start.go:182.
- **Card 25:** `hubgeom.ParentOf` exists at parentof.go:21. `ownedElsewhereRefusal`'s callers are `refuseStartOwnedElsewhere`, `claimBoardEntry`, `resumeOwnerClaim` and `resumeRefusal`. `cancelDeps.Owner` is `(machine, owner string, err error)` today.
- **Card 16:** `websterEntry` names its seams as `WebsterDeps.&lt;Field&gt;` through `requireSeam`.
- **Card 18:** a board write cannot set the beam flag (project.go:79-82). The dobby releases tasks only (`released`, starts.go:37).
- **Card 14:** the cancel writes `StatusDraft` (cancelcmd.go:461).
- **Card 2:** the PATTERN background's line 3 says "ended only through `runproc.End`".

Tool calls:
1. Grepped replay_test.go owner rows, `TestStartFirstPass`'s failed-spawn row, the `TestPrimeSegments` span, and `dobbyengine.Live` / `hubgeom.ParentOf`. My context did not show those lines or symbols.
2. Read project.go:70-95 and cancelcmd.go:455-465 to check the beam-flag write path and the draft write after a cancel. My context showed neither.
