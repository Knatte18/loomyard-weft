# Transient push failures are mailed to the orch one by one instead of retried

`push-failures-mailed-one-by-one` · Note · bug, fabric

Imported from [issue #238](https://github.com/Knatte18/loomyard/issues/238).

On a network that resets HTTPS connections now and then (git exits 128 with 'Recv failure: Connection reset by peer'), every failed background push sends the orch a creel message naming `lym fabric push` or `lym board sync` as the way forward. In one hour on one hub the orch got eight such messages, for the bolt branch main, task code branches and records branches, and each retry by hand succeeded at once or failed again only because the connection was still flaky.

Expected: fabric retries a transport failure itself with backoff, and mails the orch only once retries are exhausted. One message then covers every branch still unpushed, rather than one message per push. The next pair push already carries a withheld code side, so a single summary would suffice.

Related: #193 (closed), which made runs retry transient fetch failures; pushes have no such retry.