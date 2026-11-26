---
round: 2
exclude_lenses: []
focus:
  - "Module cut: check which packages the record counts as fabric and as above fabric, against the imports of internal/fabriccli and internal/fabricengine, and where the detached children (internal/fabriccli/teardownchild.go, the push child in internal/fabriccli/weft_verbs.go) get their seams. Does the declared PATTERN-fabric-boundary text hold for that set, and can its guard fail?"
  - "Shed records across the cut: how do the writes in internal/fabricengine/codetipsettle.go and takeoverwrite.go, and the readers in originreads.go, handover.go and takeover.go that return shed types, work without a shed import, with the same status file bytes?"
  - "Seams: how is the hub workspace regenerated inside Add (rollback included, add.go), Remove (remove.go) and the anchor change (anchorchange.go), and what does each seam, including the prime-run lock, status lock and run-records path, do when unset or failing?"
  - "Core and executor split: where do push retry classification (pushclassified.go), the backoff wait, the archive tag push's inner retry (archive.go), a mid-list failure or destroy.go refusal, Add's rollback with its ownership tokens, and the printed mutations array end up, and does verb behaviour stay unchanged?"
  - "Detached teardown: which fields of remoteTeardownJob (remoteteardown.go) are handed to the child and which does the child decide itself, does the pending-teardown record keep its shape, and does the in-process path under test follow the same rule?"
  - "Part ordering: can each part land with the PATTERN text and guards in force at that point, including result types in internal/fabricengine that embed MutationRecord or carry shed types?"
  - "Tests and speed: check the admitted test imports (internal/testkit kits such as boardkit, configengine for config-derived values), the claimed gate-scope reduction given hubforge, boardkit and envkit importing fabricengine, and whether the speed target is measured so that load cannot decide it."
  - "Unpinned tool behaviour: is what gitrepo's go-git ref reads return on a store with extensions.refStorage=reftable stated and pinned, with the fallback covering refs as well as worktrees?"
---
