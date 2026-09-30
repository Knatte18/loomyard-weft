# Rework coverage — PR #321

- **1. HIGH — `goto` can skip review and approval, and can reach teardown** → cards 32, 33, 35 (table rows in 43).
  Card 33 refuses a running run and every target past the current row (strictly before it when awaiting, which keeps finding 4b's backward goto from PR-Gate legal).
  Card 35 adds batten's Worktree-Teardown guard and drops the help example.
  Card 32 keeps Loom-Preflight's half-finished way forward working under the backward-only rule.
- **2. HIGH — fabric-reference audit findings only warn, yet can rewrite run state** → card 36 (table rows in 43).
- **3. MED — `rebaseline` compares card IDs only** → card 37 (table row in 43).
- **4a. approve HEAD-differs names `lyx fabric push`** → card 40 (table row in 43).
- **4b. approve's "lyx loom step opens it at Publish" loops** → cards 40 and 33 (table rows in 43).
- **4c. budget-exhausted and missing-producer texts omit the run-id** → card 34 (table rows in 43).
- **4d. `refusal-spec.md` says "at Publish"** → card 43.
- **4e. `missingStatusWayForward` names `lyx shed seed` for batten** → card 34 (table row in 43).
- **5. MED-LOW — a run-exit correctness halt clears on a bare re-step** → cards 38 and 39 (table rows in 43).
  This takes the finding's explicit-acknowledgement option: a content re-check cannot soundly clear a suspect `state.json` or a pathless fabric-reference finding.
- **6. LOW — stale Master stencil** → card 41.
- **7. Operator rule — the fabric side names belong only inside fabric** → card 42 (tests) and card 43 (`refusal-spec.md`).
- **Nit — `docs/overview.md` shedverbs line lacks goto** → card 43.
