# The parent-review ask record, its deadline keys, and lym loom parent-review answer and await

`parent-review-ask-verbs` · Task · feature, loom, orch, config, burler · running · Darn-Write

Decision record: [Sketch](decision-parent-review-ask-verbs.md)

Filed from the Sketch `parent-reviews-decision-record`.

Takes: the Sketch's decision record; CapAssistTimeout, the decision add --by parent guard, KeepWarm, lock and state
Delivers: the loom.yaml keys discussion_review_ask_parent and discussion_review_parent_timeout_min with validation and template comments; the never-tracked ask record and parent report under .lym, with forward-only state moves under one per-run lock; the capped-deadline function; lym loom parent-review answer and await
Exports: the ask record's read, write and state-move API in burlerengine, the deadline cap in loomengine, lym loom parent-review answer <slug> --ask <n>, lym loom parent-review await <slug>

The part changes these packages:

- internal/loomcli (about 450 changed lines)
- internal/burlerengine (about 280 changed lines)
- internal/loomengine (about 110 changed lines)
- internal/lymdirs (about 5 changed lines)
