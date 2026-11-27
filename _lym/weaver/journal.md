## Batch 01-heddle-standing-keys, attempt 1

Cards 1-3 are committed; the batch gate passed (tier2 and tier3 not run).
- Shared manifest keys are exported constants in internal/shuttleengine/manifest.go (PromptEngineKey, HandWrittenPromptKey,
  QuestionDenyKey, WebsterForkDenyKey, BatchingNudgeKey, DenyNoticeKey, CreelTrustKey, SubagentPassageKey). Codex should use them.
- Claude's other keys, which Codex must match: instruction-source is the absolute path for user files above the repo,
  the root-relative slash path inside it, and "heddle-file/<slug>" (the prefix is claudeengine's heddleFileKeyPrefix,
  so move it to shuttleengine rather than repeating it); plugin-context is "<plugin id>/<hook command>";
  standing-passage is the slug; skill-load and skill-listing are the skill name.
- InstructionItem and OmittedHook now carry Plugin and Hook; UserSetup carries ListedSkills.
- testOnlyCodeAllowed holds only the planned MissingItems entry. Card 2 needed a second, interim WriteManifest entry,
  which card 3 removed. A card that creates a function before its caller will fail the same tier1 guard.
- A fake repository in a tier0 test needs .git/HEAD, .git/objects and .git/refs, or dotgit.FindRoot skips it.

## Batch 04-codex-manifest, attempt 1

Cards 4-6 are committed; the batch gate passed (tier2 and tier3 not run).
- Card 8: testOnlyCodeAllowed holds no burlerengine.Engine.SeatTables entry, since the guard reads the method as reachable.
  The entries for MissingItems, NewWithSeams, seatengine.Specs and landingshed.FixReviewSeatTable are there to delete.
- heddleFileKeyPrefix moved to shuttleengine.HeddleFileKeyPrefix.
- shuttleparity: PrepareEach(set, cfg, stencilsDir, dir, specsByEngine), Compare(set, manifests), seatsOf(env, seatTables);
  its tests are in package shuttleparity.
- A SeatEnv needs Location, Dirs, Registry, StencilsDir, SpecsDir, Dir (stand-ins, caller removes), Shortname, Slug and ParentName.
  darn, landing, drift, loom and tapestry load strictly, so a hub fixture needs their base files (configengine.SparseBaseHeader).
- A Location outside lymcwd comes from lymcwd.NamedWorktree; a literal fails TestEnforcement_LocationLiterals.
- The default claude.yaml session_scope turns off plugins whose skills a spec does not preload, so a fixture that needs
  every plugin's hook sets session_scope.enabled: false.
- Seen once on card 4's gate, unrelated and passing on rerun: internal/shedverbs
  TestUntilStop_EnvelopeCurrency/an_envelope_the_run_has_moved_past_is_retired_and_a_fresh_loop_spawned.
