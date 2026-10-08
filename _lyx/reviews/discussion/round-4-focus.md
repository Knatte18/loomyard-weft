---
round: 4
focus:
  - "D7 record write: settle which struct the stop verb writes the run record from when the run is held in-process, and whether every `saveRunState` call on the `Run`'s state (including the held-turn `NotifiedOffset` save and the `Started` save in `Wait`) takes the same lock; check that no sentence claims a smaller shared set than the code has."
  - "D3 confirm loop: settle whether a confirm read taken after the submit window closes can send an Enter, what the sleep before each post-Enter read is, and whether D6's overshoot list covers every tail past the window, including the idle wait's last poll."
  - "Completion-signal constraint: settle whether the `stopped` value reaches the record from one function at one return, so the scanned-identifier tripwire sees both the stop verb's write and `finalize`'s write for a handle-marked run."
  - "Failure class of a send whose text never appeared: settle whether it wraps `ErrSubmissionNotLanded`, and whether D4's retry and D6's loop end treat it consistently with the clear's matching rule."
  - "D3 clear premise: settle which engines the clear applies to, whether the box is proven empty before typing for each, and whether the open risk describes what a prefix or paste-placeholder match can delete."
  - "D6 restore: settle on which tick the gate is evaluated after the override releases a cleared boundary, whether the unsent re-prompt mark is restored, and where the per-tick pane read is stated."
---

The fixes of the previous round are new text; the reviewer reads the whole decision record afresh against the rubric.
