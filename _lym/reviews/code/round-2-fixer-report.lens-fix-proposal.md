```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1 and F2, both fixed as proposed.

No findings.

- **F1:** I built the case where `wantKey: "efforts.haiku"` passes with the defect in place, and could not. The key is built as `effortsKey+"."+key`. The `UnknownEffort` row fails if `Load` drops it from the message. The row's content contains `efforts:\n  haiku:` rather than `efforts.haiku`, so no echo of the content can satisfy the check. The `return` after the `errors.Is` failure stops a nil `err` from reaching `err.Error()`. The empty `wantKey` on the other rows leaves their behavior as before.
- **F2:** The new doc sentence matches `effortFor(cfg.Agents, "", spec.Effort, spec.Model)`, an explicit effort first, then the alias entry, then the default. A bare alias such as `haiku` and an empty model are both covered by "else the default entry". An unloaded `Config{}` would yield an empty effort, but `OpenLaunch` always receives a loaded config. The sentence also sits before "It validates the effort", so the validate-then-write order stays accurate.

Tool calls made: none, since my context already held every changed text.
