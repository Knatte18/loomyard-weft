```yaml
lens: fix-proposal
outcome: reported
```

No findings.

Standalone `burlercli` (`Dirs{Anchor: res.StateDir}`) gets no `Weft`, so a Codex launch there adds no writable root. That is the intended "outside a hub" behaviour: the state directory is outside any hub and has no weft sibling. The hub callers I checked (`loomcli`, `pairteardown`, `battencli`, and the others listed in the grep) all go through `fabricengine.ConfigDirs`, so they get `Weft`. For an `AnchorRel` other than `.`, the links point into `<weft>/<AnchorRel>`, which is inside the `Weft` root the engine now adds, so the root covers them.

Tool calls (1):
1. Bash: read `burlercli/wiring.go` around line 211 and grepped the other `shuttleengines.New` callers' Dirs sources. My context showed only the call sites, not where each caller's Dirs comes from.
