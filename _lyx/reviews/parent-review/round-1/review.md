---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md D3 and A2
    summary: D3 sets the seed template default to haiku[medium], moving every worktree's seed passes to Haiku 5.5 on landing; the parent rules this task adds keys only and changes no role's default model
---

### [BLOCKING] D3 changes the seed model by default

**Location:** `_lyx/discussion/decision-record.md`, D3, A2, the Scope "Out" bullet that excepts D3, the Open risks bullet on D3, and the acceptance criterion "each seed on `seed`" where it implies a Haiku default.

**Issue:** The operator ruled on 2026-10-11, after the board entry's seed sentence was written, that the Haiku work for the Bouncer is dropped: the judge and seed together are under 4% of run weight, so there is nothing to gain, and the runs stay on Sonnet.
This task is wanted for the keys on their own merits, not to move any role to another model.
The board entry's line that the seed "moves to Haiku 5.5 on all three rows once the key exists" is stale and will be corrected on the board; it predates that ruling.
D3 would silently move the seed pass of every in-flight worktree to Haiku at its next run start, which is a model change the operator did not ask for in this task.

**Fix:** Give `seed` a template default that keeps today's behavior: empty, taking the resolved judge model of the same segment, so that with every new key empty or absent each seed runs on exactly the model it runs on today.
Drop D3 and A2, remove the D3 exception from Scope "Out" and its Open risks bullet, and state in the acceptance criteria that with every new key empty a seed runs on its segment's judge model.
Everything else in the design (D1, D2, D4–D11) is in scope and stands.
