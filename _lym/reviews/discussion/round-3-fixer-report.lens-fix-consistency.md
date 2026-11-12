```yaml
lens: fix-consistency
outcome: reported
```

Task: check, through the fix-consistency lens, whether the round-3 change set still agrees with the sibling decisions, criteria and cross-references it touches.

**MEDIUM** — new D4 criterion "a testmap call in a task pair that changed maps spawns the detached records push, and that push carries the task's code branch."
Summary: the criterion is unqualified, so it contradicts the sibling D4 criterion on the held push.
Evidence: the sibling criterion says "a testmap call in a task whose entry the run-scope origin reconcile reads as ended, with the parent lacking its work, commits its maps and pushes nothing". D4's text says the verb "spawns the push only when that reconcile's verdict lets a loom run's push go". Read literally, the new criterion requires a push in exactly the case where the sibling forbids one. It needs a qualifier such as "whose reconcile verdict lets the push go".

**LOW** — Constraints `PATTERN-owner-machine` bullet, which now ends "a hold pauses no run and posts no notice. See PATTERN changes."
Summary: the bullet now carries a hold rule that the PATTERN change it points to does not mention.
Evidence: the `## PATTERN changes` entry still reads only "a loom push, or `lym gate testmap`'s push in a task pair, reconciles run-scope origin first." The pointer "See PATTERN changes" now leads to a text that lacks the no-pause/no-notice clause. A reader of the PATTERN entry alone would not learn that the verb's hold differs from loom's stop.

Out of my lens, noted once: the new D4 interface has the caller run the reconcile and then pass its result in. Today `guard` runs the reconcile only after `skip` returns false, so under the new interface loom would run the reconcile even when pushes are off or a pause covers the run. That is a behaviour change for the fix-claims or fix-proposal lens to weigh.

Further tool calls: none. Everything I checked was already in my context.
