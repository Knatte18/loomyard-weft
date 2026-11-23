---
round: 1
focus:
  - "internal/sheddriver/drive.go wait and goDriver.settleStop: does the wait honor context cancellation, bound total time to the stated 15 s, 30 s, 1 min schedule, and avoid re-stepping a stop that is not a network failure?"
  - "internal/sheddriver/driverules.go decideDrive and networkFailure: does the rule order keep non-network transient stops on their old path, and does the hand-back after exhausted waits skip the tackler as the commit message says?"
  - "internal/gitexec/transport.go transportPhrases: do the new phrases (HTTP/2 stream close or reset, receive or send failure, early EOF, TLS cut, 502/503/504) misclassify a non-transient failure such as an auth refusal or a missing ref as a transport failure?"
  - "Scope against the board entry: it asks for retry with backoff on fetch, push and ls-remote, and for weft pushes and a landing stop; does the diff cover each surface the entry names, or only the shed driver's re-step?"
  - "Commit-message claims: check each Root-Cause trailer names a defect the diff removes, and that the notice wording the second commit describes matches what networkNotice and handBackStop produce."
  - "Tests: TestGoDriver_Settle, TestDecideDrive and TestIsTransportFailure; does the wait test avoid real sleeps, assert the wait schedule from literals, and cover the exhausted-waits hand-back? Were any existing assertions at the review base widened?"
---

The decision record holds only the board entry under `## Goal`, with no added entries.
It is the measuring stick; its other sections are empty.
