---
round: 2
ledger:
  - key: d5-watched-window-bound-unverified
    rounds: [1]
    status: resolved
  - key: buildopencmd-comment-only-in-optional-notes
    rounds: [1]
    status: resolved
  - key: ac4-unobservable-session-id-ordering
    rounds: [1]
    status: resolved
  - key: d5-bypass-driver-reach-omits-creel
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: empty-mode-session-not-guaranteed-to-prompt
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d5-hook-bound-names-hooks-lyx-does-not-install
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1's three findings were not raised again in round 2 and are resolved.

Round 2 raised three findings, all on D5 and D1.
F1 (MEDIUM) says D5 names `lyx shuttle send` as the only driver of the bypass session, while `lyx creel inject` and the `lyx creel post` reminder also type into its pane, so the stated reach and bound are wrong.
F2 (LOW) says an empty or `prompt` mode starts the session in the settings layers' default mode, not necessarily prompting.
F3 (LOW) says D5 cites hooks as a bound that lyx does not install.
