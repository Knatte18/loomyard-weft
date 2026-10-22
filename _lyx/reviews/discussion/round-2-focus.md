---
round: 2
focus:
  - "D5 in _lyx/discussion/decision-record.md: where a remove entry naming no lens is logged and how often; settle whether the mechanism keeps internal/lenses inside its import set (standard library, internal/configengine, gopkg.in/yaml.v3, pinned by internal/lenses/leaf_enforcement_test.go) and logs a standing condition once, as warnUnknownKeys in internal/configengine/config.go does."
  - "D4 in _lyx/discussion/decision-record.md: for a standalone worktree with no lenses.yaml, settle whether any scaffold path (configengine Set via scaffoldIfMissing, Edit via BaseScaffold, standalone reconcile) can leave template-equal base prose in the file so that lens_set picked through lyx config lenses --set no longer resolves the overlaid lenses to the picked layer."
  - "D3 in _lyx/discussion/decision-record.md: for a lens_set key present in a worktree file above a hub base, settle whether the named way forward (lyx config lenses --layer default --set lens_set=<name> from the prime) leaves the worktree key in place, and whether the worktree editor path (configengine Edit, no load check) or a tracked-file merge can reach that state; also settle which message the local layer gives, since lens_set is a Guard key refused by ValidateLocal."
  - "Any text the round-1 fixes add to _lyx/discussion/decision-record.md: check each new Decision, Constraint or acceptance criterion against the attack-surface question and against the deferred-research-question rule, and check that no criterion claims a reconcile drop that no code path performs for a hub worktree file (reconcileSparseFile in internal/configsync/configsync.go only migrates and prunes)."
---

The focus names where the round-1 fixes land; it caps no severity and pre-states no verdict.
