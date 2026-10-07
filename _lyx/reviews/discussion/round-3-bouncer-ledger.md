---
round: 3
ledger:
  - key: round-gate-tagged-compile-gap
    rounds: [1]
    status: resolved
  - key: publish-verify-unbounded-config
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: lyx-guard-missed-marker-unbounded
    rounds: [1]
    status: resolved
  - key: comment-lint-flags-untouched-lines
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: comment-lint-terminator-rules
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: NIT
  - key: diff-mapping-rename-delete-baseless
    rounds: [1]
    status: resolved
  - key: empty-set-no-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: verify-record-retention-and-lookup
    rounds: [1]
    status: resolved
  - key: round-command-length-growth
    rounds: [1]
    status: resolved
  - key: tier-green-ordering-misplaced
    rounds: [1]
    status: resolved
  - key: decision-record-narrative-duplicates
    rounds: [1]
    status: resolved
  - key: publish-verify-no-publish-landings
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d2-llm-catch-claim
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d7-no-base-lint-premise
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-wrong-gate-catcher
    rounds: [2]
    status: resolved
  - key: d8-guard-tag-set
    rounds: [2]
    status: resolved
  - key: d5-guard-enumeration
    rounds: [2]
    status: resolved
  - key: d6-pattern-bullet-clause
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: goal-universal-claims
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eleven findings; most were resolved by round 2.
Round 2 re-raised three of them and added six.

Round 3 raised eight findings, mapped as follows.
F1 (BLOCKING design) is `publish-verify-unbounded-config`: the key reaches the parent's `landing.yaml` only by a hub re-apply, so the "everything reaching main" claim is false.
F2 is `publish-verify-no-publish-landings`, now a MEDIUM consistency finding: D2's bound omits landings that run no Publish verify.
F3 is `d2-llm-catch-claim`: "one rework round" and "runs every test" do not match the recipe or D4.
F4 (MEDIUM design) is `comment-lint-flags-untouched-lines`: D7's checked-line scope cascades through a legacy block under D6's rewrap rule.
F5 (MEDIUM design) is `d7-no-base-lint-premise`: only two fallbacks leave no base, and Webster-gate fix commits are linted by no gate.
F8 (NIT) is the colon sentence end stated twice, carried on `comment-lint-terminator-rules`.
F6 and F7 are new LOW keys.

The round 2 keys `d5-wrong-gate-catcher`, `d8-guard-tag-set` and `d5-guard-enumeration` were not raised again and are carried as resolved.
