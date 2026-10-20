# Verify runs the unit tests twice: the integration tier already includes them

`verify-unit-tier-twice` · Note · enhancement, config, shed

## Seen (orch, 2026-10-20)

The verify command of `darn.yaml` (and the loom verify gates) is `lyx gate test ./... && lyx gate test --tags integration ./...`.
`go test --tags integration` builds and runs every untagged test as well as the tagged ones, so the unit tests run twice, and on OSL-1046 with Cortex XDR each tier also links its own set of test binaries that XDR holds 8-10 s each.

## Idea

- Verify runs only `lyx gate test --tags integration ./...`, or the first tier stays only as a fast fail when it is measurably faster to fail there.
- Check first whether any test is excluded by the `integration` tag (a `!integration` constraint), which would make the two runs differ.
