# Timing an upkeep-only tier1 test through lym gate test

The prompt's own gate key, `darn-write`, skips a test marked `//lym:only upkeep`, so the before and after timings the description needed took a round trip to the parent.
Then `lym gate test --gate upkeep ./internal/gatecli -- -run ...` ran the tier1 test under tier0, where it skipped with exit 0.
Only the same call with no package argument ran it under tier1.
`--tier 1` beside `--gate upkeep` is refused.
