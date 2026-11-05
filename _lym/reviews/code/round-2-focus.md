---
round: 2
focus:
  - "internal/configsync/configsync.go ReconcileHubWideAt seed-only branch and internal/yamlengine/follow.go FollowTemplate: when the template base file is unparseable or not a YAML mapping, does the reconcile fail naming the base file and the way forward, and does a later reconcile recover instead of failing the same way on every start verb? Settle whether the error names the hub file or the base."
  - "internal/configcli (lym config reconcile --layer default --apply), internal/fabriccli/fabric.go reconcile and internal/fabriccli/clone.go: is the template base committed on Bolt beside the config file on each of these paths, and does a test reach each path?"
  - "internal/yamlengine/follow.go re-encoding: does a followed file keep a flow mapping longer than 80 columns on one line, and do blank lines between entries survive or does the doc comment state they do not?"
  - "internal/hubreconcile/walk.go followedLines and notifyTemplateFollowed, and internal/configsync/configsync.go readTemplateBase with the restore of a base written before a later failure of the pass: which branches (added, updated, kept-the-hub's, no mail, unreadable base, restore) are reached by a test?"
  - "internal/configcli/hubwide.go reconcileDefaultLayer doc and internal/configcli/doc.go: does any text still say a present board file is never filled while a seed-only file follows its template?"
  - "internal/configsync/configsync_test.go: does every test function in the file sit directly under its own doc comment?"
---

Focus for round 2 after a round with two gating design findings on the template base.
