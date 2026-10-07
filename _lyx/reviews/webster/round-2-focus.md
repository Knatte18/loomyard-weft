---
round: 2
focus:
  - "Comments this branch added or changed in internal/fabricengine/checkout.go (around lines 103, 137-146, 191), internal/fabricengine/mutation.go (around 65-67), internal/fabricengine/reconcile.go (around 5-8, 41, 526) and internal/landingshed/finalize.go (around 164, 218-221, 261): does each changed sentence sit on its own line with no mid-clause wrap and no two sentences on one line, as PATTERN-comment-line-breaks requires? Judge against the current HEAD text, and check any comment the newest commits touched for the same defect."
---

Only the changed sentences are in scope; untouched legacy-wrapped lines are exempt.
