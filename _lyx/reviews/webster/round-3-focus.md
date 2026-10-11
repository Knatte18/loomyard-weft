---
round: 3
focus:
  - "In internal/reedengine/socketfile_test.go, the stale-socket rows of TestRemoveStaleSocket (row 'stale socket is removed') and the gone-server row of TestRemoveSocketFileOnceGone build a stale socket file for removeStaleSocket, which keeps any path that still accepts a net.Dial. Settle whether each fixture now yields a path that refuses Dial for every fd holder, including a child forked by a parallel test before exec, on Linux and with Windows still compiling (build tags). Check the production removeStaleSocket and its doc comment are unchanged in behavior."
  - "Publish verify failed on TestRemoveStaleSocket/stale_socket_is_removed in the full-repo run (log .lyx/verify/verify-4.log). Settle whether the fixture change in the branch's latest commit (ea4e53819, stale-socket rows bind a datagram socket) fully removes the dependence on listener fd inheritance, and whether any other test in internal/reedengine closes a listener and then expects Dial to fail."
---

Round 3 reviews the branch's fixture change fresh, alongside the branch's own range.
