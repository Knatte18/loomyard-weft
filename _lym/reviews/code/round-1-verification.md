---
round: 1
entries:
  - key: fabricsync-calls-not-gofmt-clean
    status: closed
    finding: F1
    reason: "204d20239 adds the space after c.syncOptions, at the fabricSync calls in all nine named files; gofmt -l internal/webstercli internal/loomcli prints nothing and no unspaced call remains."
---
