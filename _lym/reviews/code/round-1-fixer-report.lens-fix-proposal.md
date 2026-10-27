```yaml
lens: fix-proposal
outcome: reported
```

no findings

- F1 (`render.go:4`, `render.go:238`): the new list "Running, Ready, dependency layers, Independent and Draft" matches `ComputeLayers` in `layer.go:3`, and `layerSection` has an `isolatedLayer` case and a `draftLayer` case.
- F2 (`render.go:442`): `standingCell` returns each name the new list gives: `Done` when finished, `Note` for a note, the run's place for a running task, `Ready` for a ready task, `Project` through `kindName(KindProject)` for `projectLayer`, and a layer name through `layerSection` for everything else; for a draft that is `Draft`.
- F3 (`doc.go:176`, `layer.go:29`): "open, released tasks" fits the `layerSection` text "Not released; nothing starts it until its status is cleared". "An open task that is not a draft and has no open dependency is readyLayer" fits the code at `layer.go:42`, where `isDraft(t)` is tested before the ready layer. A running draft cannot exist, because a draft holds no run lock; the review rejected that case for the same reason. The `layer.go:29` sentence therefore does not contradict the `layer.go:27` sentence above it.

Tool calls used: 1.
- `sed -n 440,475p render.go; grep -n ... layer.go`: to read the `standingCell` body, `layerSection` and the `layer.go` draft branch, which my context lacked.
