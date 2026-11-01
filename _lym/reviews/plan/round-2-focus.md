---
round: 2
focus:
  - "Cards 10, 13 and 18 create the new test packages internal/entrymerge, internal/heddleengine and internal/heddlecli: settle whether each package gets a TestMain through tmuxkit.Main (and gitkit.HermeticGitEnv where its tests spawn git), as the tier0 guards in cmd/lym/tmuxisolation_test.go and cmd/lym/hermeticenv_test.go require."
  - "Card 2 gives internal/runproc an internal/lymcwd import for HubOwner, and internal/runproc/leaf_enforcement_test.go lists allowedImports: settle whether the card keeps TestLeafInvariant_AllowlistOnly passing at its own gate."
  - "Card 18 (internal/heddlecli) against PATTERN-cli-cobra: settle whether the module exposes Command(), RunCLI(out, args) int and RunCLIIn(cwd, out, args) int like the other cli modules, and whether the Intent states what lym heddle set and remove let through and what bounds them."
  - "Card 7's TestEnforcement_BoltDirNameDeclaredOnce reads files from scankit.Root outside its own package: settle whether it carries //lym:guard on the line above its func line, as PATTERN-test-speed requires."
  - "D9 rules that anything in Bolt beside the board takes its path from BoltDirName: settle whether production callers outside internal/fabricengine that build Bolt-checkout paths from fabricengine.BoardDir (for example internal/tendercli/gate.go, internal/orchroute/paths.go, internal/hubgeom/creelgeom.go) are carried by some card."
  - "Card 9 and Fabric.FastForwardPair in internal/fabricengine/parenttips.go move a checked-out code branch to origin's commits: settle whether the plan says if D6's took-in event fires there or D6's landing exception covers it."
  - "Card 8's Topology.Reconcile re-seed row against repairPairWiring in internal/fabricengine/reconcile.go, which re-wires only a pair whose junctions are unhealthy: settle whether the row's input reaches the seeding."
  - "Any Added after Discussion entries now in _lym/discussion/decision-record.md against the plan's run-ledger Shared Decision (card 12) and card 18's hub resolution: settle whether every departure from a Decision or Constraint is licensed by an entry."
---
