```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set — checked the set-premise findings (F2, F26, F13/F5) by rerunning their searches. No findings.

- **F2:** no production site still writes the beam module name as a literal. The remaining `"beam"` hits are CLI verbs and args, log keys, unrelated constants (`lymdirs.BeamDirName`, `ShapeBeam`, `HubPullBeam`), the board's `"beam"` project flag, and `beamsetting.Module` itself.
- **F26:** no stale references remain to `settingPlaces`, `settingPlace` or the prime root/anchor parameters of the reinterpretation. The `CheckDirs` signature in `internal/configreg/configreg.go:84` is a different hook and unaffected.
- **F5/F13:** no references to `LegacyBeamSettings` or the `keepOff` parameter remain in Go or markdown. The only doc mention of the retired keys is `internal/boardengine/doc.go:123-124`, which the fix rewrote.

Findings left unchecked: none.

Calls made:
1. Grep of production `.go` files for `"beam"` literals, the stale `settingPlace`/prime-root names, and `LegacyBeamSettings`/`keepOff` — my context did not hold the repo-wide state of the F2, F26 and F5/F13 sets.
2. Grep of `.md` and `.go` files for `notify_on_land` and "retired entry keys" — to catch stale docs outside the files already shown.
