# Card 24 — gate-tier-docs

**Prosa:**
- `pattern/PATTERN-test-speed.md`
- `PATTERN.md`

**Intent:**
Record the gate model this plan built, now that every site and route exists.
`pattern/PATTERN-test-speed.md`'s tier table and gate table state: the card gate runs tier 1 of the card's own packages through `lyx gate test`, then the comment lint (D1); the batch gate runs `lyx gate test --tags integration` once per batch over the batch's packages (D2); Webster's gate, Publish and Finalize run the plan's `## verify:` in full, and Publish then `publish_verify`, the `tmux` tier (D3); the Webster-Burler round gate runs its impacted-set command, plus the failing tests and an impacted-set `tmux` pass only while a Publish failure record is present (D11).
It records as a decision that no gate runs the `llm` tier: every gate compiles it through `go vet -tags llm`, and running it is the operator's call, because an `llm` run is billed, nondeterministic and needs a live session (D14).
It states that every gate build or test run takes a hub slot, and that agents run module-wide and `tmux`-tier tests only through `lyx gate test`.
`PATTERN.md` gains, in the Shed and loom section, the slot entry: running a gate build or test from Go or an agent goes through a slot of `internal/gateslot`'s hub-wide pool, by `verifytree.Verify`, the card-verify rerun, the plan gate's type load or `lyx gate test`, with `-p` capped, and no agent starts a raw module-wide `go` run.
The `PATTERN-test-speed` entry's line mentions the gate table only if its wording no longer holds.
