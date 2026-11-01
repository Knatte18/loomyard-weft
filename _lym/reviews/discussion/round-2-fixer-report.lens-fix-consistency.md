```yaml
lens: fix-consistency
outcome: reported
```

Checking fix-consistency: whether the decision-record edits for F1–F5 agree with the sibling decisions, criteria and cross-references they touch.

Twin and sibling checks:
- F1 against D2's line 48 sentence, D2's rationale, D3's machine-layer bullets, D4's failed-desktop-load bullet and the D2 criterion.
- F2 against D7's confirm-before bullet and the D7 criterion.
- F3 and F4 against D8's rationale, D4's run-directory clause and the new Auto-mode assumptions line.
- F5 against D7's identity sentence and the Bound.

Findings:

1. MEDIUM. Changed text: the new D2 sentence "In desktop's machine layer a group written as a mapping is refused as a key the template lacks, so desktop's load fails, adds no machine buttons (D3) and fails that load's `afterWindow` report."
   Summary: it contradicts the sentence just before it in D2, which the fix left as it was.
   Evidence: `decision-record.md` D2 still says the render "skips one of the wrong YAML type with the same Warn as any other invalid entry, so such a type error stays inside the button row and leaves whole every load of `reedengine.LoadConfig` and of `desktophook.LoadConfig`, whose `afterWindow` runs on every window lym opens." A group written as a mapping is a group of the wrong YAML type. In the machine layer, the new sentence says it fails `desktophook.LoadConfig` and `afterWindow`. Line 48 needs a carve-out for the machine-layer mapping group, or it should claim only the types the layers admit.

2. LOW. Changed text: the same new D2 sentence.
   Summary: D2's rationale "A bad button must not stop agent spawns, so it costs only itself" is now stale.
   Evidence: a machine-layer group written as a mapping fails the whole desktop load. That costs every machine button and the `window_command` run by `afterWindow` (`internal/desktophook/window.go:44-47`), not only the bad button. Agent spawns are still unaffected, so only "costs only itself" needs a qualifier.

All other twins agree:
- D3's "mapping or not, admits a non-empty list" and "A failed desktop load … adds no machine buttons" match the new D2 text.
- D8's rationale and D4's run-directory clause match the rewritten D7 run-directory bullet and the new Auto-mode assumption.
- The D7 criterion "no click, refused or failed included, opens a view-mode pane" now holds under the forced zero status.
- The Bound's existing "--confirmed skipping its prompt" sentence agrees with the added confirm-path sentence and the narrowed identity clause.

Further tool calls: none. My context already held the changed record and `window.go`.
