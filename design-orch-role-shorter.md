# Shorten the orch role file, the largest single start cost

`orch-role-shorter` · Note · enhancement, orch

## Seen

Report `docs/research/tokens/startup-context-2026-11-05/report.md`: `.lym/orch/role.md`, 54,443 characters, costs 19,390 tokens each time it is read, which is every orch start. It is the largest single source in an orch start, larger than everything Claude Code itself loads.

## Idea

Shorten `contracts/orchrole`, or move the parts the orch reads only for a rare verb or event (such as intake, beams, landing halts) into a skill or a file it reads on demand. The command list alone is a large share and repeats what `lym <command> --help` gives.