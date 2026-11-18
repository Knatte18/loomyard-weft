---
round: 2
exclude_lenses: []
focus:
  - "decision-record.md D8: where a hub agents.yaml names a role the record deletes, check that internal/agentconfig Load (config.go, skillLists) still loads it, and that any rewrite ships as a configreg.Reinterpretation applied by hubreconcile per PATTERN-config-layers; settle whether the acceptance criterion holds as written."
  - "decision-record.md D4: check that every test invocation of the live gate names a hub gate.yaml gate key and no tier list, per PATTERN-test-speed (tests run via lym gate test by gate key); settle what each key pins, including any claim that a run is serial against the hub's tier-2 parallel setting."
  - "decision-record.md D4 --copies: check against internal/gateslot/doc.go (nested gate run inherits its parent's slot; multi-slot acquire waits for every slot) that the concurrent copies actually get their slots, and that the stated shortfall bound describes a grant the pool can make."
  - "decision-record.md D5 and the PATTERN changes section: check that every process the sandbox reaps is ended through runproc, or through EndProcess only within the declared PATTERN-child-processes change (the Hardener sandbox reaper beside lym tender reap, bounded to processes under a hardener run's sandbox root that no record names); settle the bounds for a reed server whose hub is gone, a hub outside the root, and teardown of a root holding repositories."
  - "decision-record.md D5, D6 and D7: trace one round's stray report from every reap point through the archive copy; settle which row writes which file and whether each reap's output reaches the archive."
  - "decision-record.md D3 and D9: walk a campaign whose reviewer list has one entry or adjacent duplicates, and a round past max_bounces; settle whether CONVERGED stays reachable and who refuses an unreachable list."
  - "decision-record.md D4 and D6: walk a red live gate whose cause the fixer defers as outside the module; settle the live gate's attempt count, its source in hardener.yaml, and the outcome when attempts run out."
  - "decision-record.md D10: for the loosened order clause of the loom contract check, walk a segmented build row not reached from its segment's Bouncer and one whose segment holds no Bouncer; settle whether each is refused and where the acceptance criterion says so."
  - "decision-record.md D3 and D9: settle which source HardenFix's verify gate takes, and how the segment-budget projection used by lym loom status gets the Harden-Review cap."
---

Every entry restates its own sites.
