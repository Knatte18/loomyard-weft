# webstercli standalone tests fail when an empty /tmp/.git exists

`webstercli-tests-ignore-tmp-git` · Task · bug, webster, testkit · running · PR-Review

Imported from [issue #862](https://github.com/Knatte18/loomyard/issues/862).

Seen 2026-10-24 on Neve in the gate-runs-once run: an empty directory /tmp/.git (owner knatte, created 11:57, origin unknown, possibly a leaking test or tool running git init in the temp dir) made internal/webstercli's standalone wiring tests treat /tmp as the repo root; 8 tests failed with "derived state directory ... lies inside the standalone target /tmp".

Workaround: the orch removed the empty /tmp/.git by hand.

Expected: the tests build their fixtures in t.TempDir() without walking up into a parent repo root they did not create (or set GIT_CEILING_DIRECTORIES), and whatever created /tmp/.git should not write outside its own temp dir.