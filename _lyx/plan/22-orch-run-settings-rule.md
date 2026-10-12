# Card 22 — orch run settings rule

**Prosa:**
- `contracts/stencils/orch/orch-template-role.md`

**Intent:**
Replace the orch's hand step for a small task's review budget with run settings (D10).
The rule "A small task gets `review_max_bounces: 2` in its pair's loom config right after the pair is created; never 1, since a second round reviews the first round's fixes." becomes: a small task gets `run_settings` `{"loom": {"review_max_bounces": 2}}` on its board entry, through `lyx board upsert`, before its batten run starts; never 1, since a second round reviews the first round's fixes.
The rewrite drops the pair-exists check the old step needed, which `orch-split` needs gone for a mechanical driver, and names no command the agents' settings deny (`PATTERN-no-denied-recovery`).
One sentence per line, as the file already is.
