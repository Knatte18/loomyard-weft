```yaml
lens: fix-claims
outcome: reported
```

no findings

Calls made:
- Read `function Is(` candidates in 2.1.312 to check the claim that the summary row shows a longer claim's first 5 lines. My context had only `maxLines:eu` and `eu=5`. None of the three top-level `Is` definitions is the renderer's clamp helper, so whether the clamp spends a line on an ellipsis marker is still unverified. That is not a finding.
- Read the Stop additional-context push in 2.1.311 to check that the guard is the same in other installed versions. My context showed the guard only in 2.1.312. 2.1.311 has `if(!Cgt(xe))Ke.push(xe)`.
- Read `function Cgt(e){` in 2.1.311 to confirm the guard function's body. It is identical to 2.1.312's `Ugt`, starting `if(!Sr())return!1`.
- Searched 2.1.310 for `type:"hook_additional_context",content:` to find its Stop push site. The first four hits were SessionStart, Setup and PostToolUse sites, not Stop.
- Searched 2.1.310 for `?"SubagentStop":"Stop",` to find its Stop push site. It has the same shape, `if(!fpt(Ee))We.push(Ee)`. I could not read `fpt`'s body within the call budget.
