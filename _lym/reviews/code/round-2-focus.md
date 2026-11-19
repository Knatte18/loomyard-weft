---
round: 2
focus:
  - "internal/verifytree/reach.go `ReachTurn.findings` and `WithReachTurn`: does a function listed in one turn stay listed across later turns, so it never brings a second turn at the same hash, and is a re-run after a fixer, including the live-record path of `rowWrap.spawning` in internal/gatefixer/rowgate.go, marked as a re-run?"
  - "internal/gatecli/triage.go `addTriage` in the prime, and internal/loomcli/wiring.go `wire`: does a test drive the prime's no-change-set triage line, and does one assert that `weaverGeom.ReachPending` and `Env.VerifyReach` are set after `wire`?"
  - "internal/testreach/index.go `readIndex` and internal/gatecli/reach.go `refreshMaps`: does a reader leave any file in the shared maps directory, and does a failed HEAD read still name a reason per unrefreshed package?"
---

Focus restates the sites where round 1 found gating defects, so round 2 checks the fixes at those sites with fresh eyes.
