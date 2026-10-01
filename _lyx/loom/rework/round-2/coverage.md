# Rework coverage — PR #321, round 2

- **A. MED — the fabric-reference classifier fails open** → card 44.
  It inverts the per-program rule to an allowlist on the existing parser, and moves the `lyx FABRICREF sync` fixtures that stood for a read-only reference to `cat`, so the policy-path tests still reach that path.
- **B. LOW — accept-audit's fabric-sync-failure refusal has no spec row or test** → card 45.
- **C. LOW — pending audit findings fail the run instead of blocking it** → card 46.
