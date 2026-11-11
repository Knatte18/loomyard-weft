# gate test: card gate and explicit target never reach a second test project when tier0 command lacks a targets placeholder

`gate-target-dropped-silently` · Task · bug, gate, dotnet · running · Drift-Verify

Imported from [issue #105](https://github.com/Knatte18/loomyard/issues/105).

## What happened

A card created tests in a second test project (a slow-tests project), but the hub's tier0 command runs only the fast test assembly, and tier1 is empty.
`lym gate test --gate card --card <card>`, the batch gate, and an explicit `lym gate test <project> -- --filter ...` all ran the fast suite instead.
The tier0 command holds neither `{targets}` nor `{test_targets}`, so the requested target was silently ignored.
The card was committed after only a compile check of the slow-tests project; its tests never ran through any gate route.
The tests were later found failing only because an agent ran them through `lym gate test --tier 1 --card <card> -- --filter ...` by hand.

## Why it is abnormal

A gate that is asked for a named target and cannot pass it to the command reports a pass for different tests, with no warning.
A card whose tests live outside the configured tier0 assembly gets no verification and nothing says so.

## What to do

- Refuse or warn when a gate or explicit target is requested and the tier command has no `{targets}` or `{test_targets}` placeholder.
- Warn when a card adds or edits tests in a project that no configured tier command reaches.

Provenance: task `trip-history-transfer`; notes `04-aligner-moving-junction-recovery.md`, `04-aligner-moving-junction.md`.
