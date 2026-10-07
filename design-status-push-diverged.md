# loom: status push can hit a diverged remote mid-run

`status-push-diverged` · Note · bug, loom

Imported from [issue #418](https://github.com/Knatte18/loomyard/issues/418).

Seen in bugfix-sessions (drive report 20261006-121941): during Discussion-Burler round 1, loomcli logged `status push failed ... remote diverged`. The next transition pushed fine, so the run recovered, but the failure was logged and the status was briefly unpushed. Distinct from #404 (Publish push). Seen once.