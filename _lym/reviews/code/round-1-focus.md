---
round: 1
focus:
  - "internal/shuttleengine/send.go `confirmPasted`: does a landing send that read its box empty keep returning nil on every exit (window closed, read cap, no readable box, later box text), and does a non-landing send still return nil on an empty box at once? Check the `submitted` flag against the old `box == \"\"` branch for a lost exit, such as an empty read on the final allowed read."
  - "internal/shuttleengine/send.go: after `submitted` is set, can any path still send a key (guardedEnter) or fail with ErrForeignBoxText, ErrSentTextInDraft or failUnlanded? D2 says none."
  - "internal/shuttleengine/session_test.go: do the new tests cover each acceptance criterion for D1/D2 (empty then other text succeeds with no further key; empty then session start succeeds; foreign text before any empty read still fails ErrForeignBoxText)? Do they run on the injected clock and fakes, and do they duplicate an existing confirmPasted test?"
  - "internal/creelengine/deliver.go `probe` and internal/creelcli/delivery.go: with `PaneMail` and `hasHookDelivery` gone, does every strand without CreelHooks reach the notice and reminder path, and a CreelHooks strand only the wake or bell? Check for leftover references to `PaneMail`, `hasHookDelivery` or the provider lookup anywhere in the repo, including testkit fakes, docs and smoke tests, and for an import left unused or still needed."
  - "Duplicate delivery from D3: a Claude strand without CreelHooks may claim at a tool boundary through an older hook while a notice is typed. Does the diff leave the reserve-before-type and reserve-before-hand-out ordering intact, and do agent and program mailbox reminder rules stay as before?"
  - "internal/creelengine/deliver_test.go, internal/creelcli/delivery_test.go: do the tests cover the D3 criteria (strand without CreelHooks gets notice and reminder; CreelHooks strand gets neither and is rung or belled; hub delivery lists a Claude strand without CreelHooks for the pane path and one with it for the wake)? Are removed or rewritten tests a loss of coverage?"
  - "Docs landing with the code: the `doc.go` of shuttleengine and creelengine, any creelcli doc, and docs/overview.md. Do they state the new behavior and drop the pane-mail and provider-lookup wording? Check the commit messages, and any summary claim that both #897 and #899 are closed, against the diff."
---

Seed focus from reading the two commits, the board entry and the decision record; the review itself is the next step.
