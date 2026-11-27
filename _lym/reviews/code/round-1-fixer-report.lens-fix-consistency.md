```yaml
lens: fix-consistency
outcome: reported
```

Task: checked that the fix change set agrees with its surroundings (fix-consistency lens).

**LOW: `internal/burlerengine/doc.go`, "The fixer fan (twill fixer)", lines 110–112**
The added line contradicts the sentence two lines above it.
- Line 110 says "only the self-check forks are held to the read-only rule".
- Line 112 says "Whatever a self-check fork spawns is held to the read-only rule with it". That includes fresh subagents under a self-check fork, which `auditFixRound` now checks and which are not self-check forks.
- Dropping "only", or wording line 110 as "the self-check forks, and whatever they spawn, are held to the read-only rule", would remove the clash and make line 112 unnecessary.

I found no other inconsistency among:
- the `auditFixRound` comment;
- the `ForkAudit` and `ForkReport` docs;
- the `AuditForksIncremental` doc;
- the self-check stencil line;
- `_lym/landing/summary.md`;
- the `cluster_test.go` header.

`contracts/stencils/twill/twill-directive.md:46` ("the writer is the only session that spawns") is not a stale twin, because it covers a writer's lens review, not the fixer.

Tool calls:
- Grep for spawn and audit wording across `internal`, `contracts` and `docs`: my context had not searched for twin sentences.
- Diff stat, `cluster_test.go` header, `auditFixRound` comment and `summary.md`: my context lacked the header and the post-edit summary as a whole.
- Read `twill-directive.md` lines 1–50: my context had only its line 46.
