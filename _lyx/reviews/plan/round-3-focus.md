---
round: 3
focus:
  - "Card 8's landed check for an unconfirmed injection: decide what marks `PhaseInjected` true. Is it only a turn end past the entry offset, or a positive running-turn signal, rather than any failing `SessionIdle` probe? The case to test is a `clearing` timeout (the idle probe was already failing) followed by a resume `Send` error: does a later tick re-send once the pane reads idle? Also check whether card 8's test list drives that path."
---

The fix to round 2's F1 introduced this check.
D10 counts only a turn in progress as evidence of landing, while card 3's idle classifier also fails on permission prompts and on panes it cannot frame as an idle box.
