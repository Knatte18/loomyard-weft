```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for content in the decision record and support log that should move or be cut, applying the completeness-before-leanness test. No tool calls were made.

- **LOW — Scope "Out", D2, D4 and D8: one fact stated twice.** "Out" restates as exclusions what D2, D4 and D8 already decide and bound: inject, interrupt, hold, withdraw and forward across hubs, `post --interrupt`, and program and EXTERNAL senders.
  - **Evidence:** "Out" lists "Inject, interrupt, hold, withdraw and forward across hubs, and `post --interrupt` to another hub" and "Program and EXTERNAL senders across hubs". D2 refuses program and EXTERNAL posts, D4 refuses forwarding to another hub, and D8 refuses inject, interrupt, hold, release and `post --interrupt`.
  - **Fix:** cut these two bullets to one line pointing at D2, D4 and D8, or keep them and drop the bounds from the decisions.
  - **Gap:** `withdraw` is named only in "Out". No decision refuses it. Either add it to D8's refusal list or drop it from "Out", since the CLI has no withdraw verb.
- **LOW — Constraints, the PATTERN bullets.** Several bullets repeat what a decision already says:
  - The `PATTERN-config-layers` bullet repeats D5 (module, registry row, classified key).
  - The `PATTERN-child-processes` bullet repeats D2's "as `internal/creelcli/spawn.go` does".
  - The `PATTERN-lym-files` bullet repeats D1's location sentence.

  Keep them, because they show each PATTERN entry was checked. They could be cut to the entry name plus the clause the decision does not already carry, such as the declared-list file for daemon-wakeups.
- **LOW — Constraints, first bullet: creel facts beyond what the design uses.** "Ids per hub that are never reused" carries D3's duplicate keying and must stay. The socket and `SO_PEERCRED` facts are premises of D3's bounds and also stay. Nothing here fails the completeness test, so no cut.
- **LOW — "Auto-mode assumptions" overlaps the support log's question ledger.** The section is required by the spec and is short. Its fifth bullet (registry written at daemon start) is the only one that is not also a decision's headline; the rest name a decision. It is fine as is, and could be cut to the D-ids and the one-line choices.
- **LOW — D2's last bullet is an implementation detail.** "The worker holds no store lock while a `deliver` call is in flight" is a deadlock-avoidance hint, not a design choice. It is a requirement the plan writer needs, so it must not go to the support log. It fits better under "Notes for the plan writer", or as a one-clause bound on D3's step 4.
- **LOW — D6 rationale and D7 rationale are justification prose.** Each is one sentence and carries why the stamp position is creel-only and why the trust sentence must change. That is needed context. No cut.
- **LOW — Open risks, second bullet (`sandbox-closed-area-build`).** It is already stated as a bound in D3 ("which the closed area of `sandbox-closed-area-build` is what would close"). The open risk adds only that a broker route would be needed. Merge the broker clause into D3's bound and drop the risk, or keep the risk and shorten D3's bound to a pointer.
- **Outside my lens:**
  - `relevant.md` has a typo: "the entry D's PATTERN change edits".
  - D5's acceptance bullet assumes a machine-layer file holding a non-Machine key is refused by "the template classification test". That is unverified; another lens should check how `configengine` treats a Guard key in `.machine.yaml`.
  - The support log's Turn 2 restates every decision in one line. That is harmless, since the plan writer never reads it.

Further tool calls: none.
