loom halted failed at Publish

```
landingshed: Publish: query existing pull request: Get "https://api.github.com/repos/Knatte18/loomyard/pulls?base=main&direction=desc&head=Knatte18%3Alogging-cleanup&sort=created&state=all": dial tcp: lookup api.github.com on 127.0.0.53:53: read udp 127.0.0.1:53455->127.0.0.53:53: i/o timeout
```

producer: Publish
state: failed
anomaly: producer-hard-failure
history_entries: 46
trace_file: /home/knatte/Code/loomyard-LYXHUB/logging-cleanup/.lyx/logs/trace-20261013T112014Z-cd851089f04d9419-1344565.log

history:
- Preflight / done / 2026-10-13T07:39:12Z
- Loom-Preflight / done / 2026-10-13T07:39:16Z
- Discussion-Write / done / 2026-10-13T07:56:06Z
- Discussion-Bouncer / stuck / 2026-10-13T07:56:40Z
- Discussion-Burler-Review / stuck / 2026-10-13T08:02:30Z
- Discussion-Burler-Fix / stuck / 2026-10-13T08:03:58Z
- Discussion-Bouncer / stuck / 2026-10-13T08:04:28Z
- Discussion-Burler-Review / stuck / 2026-10-13T08:07:21Z
- Discussion-Burler-Fix / stuck / 2026-10-13T08:08:57Z
- Discussion-Bouncer / stuck / 2026-10-13T08:09:38Z
- Discussion-Burler-Review / stuck / 2026-10-13T08:11:35Z
- Discussion-Burler-Fix / stuck / 2026-10-13T08:12:40Z
- Discussion-Bouncer / awaiting / 2026-10-13T08:13:16Z
- Discussion-Bouncer / stuck / 2026-10-13T08:14:28Z
- Discussion-Burler-Review / stuck / 2026-10-13T08:17:50Z
- Discussion-Burler-Fix / stuck / 2026-10-13T08:18:43Z
- Discussion-Bouncer / awaiting / 2026-10-13T08:19:19Z
- Discussion-Bouncer / done / 2026-10-13T08:20:33Z
- Plan-Write / done / 2026-10-13T08:48:33Z
- Plan-Bouncer / stuck / 2026-10-13T08:49:02Z
- Plan-Burler-Review / stuck / 2026-10-13T08:55:27Z
- Plan-Burler-Fix / stuck / 2026-10-13T08:57:58Z
- Plan-Bouncer / stuck / 2026-10-13T08:58:27Z
- Plan-Burler-Review / stuck / 2026-10-13T09:02:37Z
- Plan-Burler-Fix / stuck / 2026-10-13T09:09:57Z
- Plan-Bouncer / stuck / 2026-10-13T09:10:28Z
- Plan-Burler-Review / stuck / 2026-10-13T09:15:53Z
- Plan-Burler-Fix / stuck / 2026-10-13T09:22:16Z
- Plan-Bouncer / awaiting / 2026-10-13T09:22:51Z
- Plan-Bouncer / stuck / 2026-10-13T09:27:13Z
- Plan-Burler-Review / stuck / 2026-10-13T10:12:51Z
- Plan-Burler-Fix / stuck / 2026-10-13T10:14:31Z
- Plan-Bouncer / awaiting / 2026-10-13T10:15:19Z
- Plan-Bouncer / done / 2026-10-13T10:16:34Z
- Batchifier / done / 2026-10-13T10:16:38Z
- Webster / done / 2026-10-13T11:02:10Z
- Webster-Bouncer / stuck / 2026-10-13T11:02:36Z
- Webster-Burler-Review / stuck / 2026-10-13T11:07:11Z
- Webster-Burler-Fix / stuck / 2026-10-13T11:07:26Z
- Webster-Bouncer / done / 2026-10-13T11:07:48Z
- Describe / done / 2026-10-13T11:08:26Z
- Merge-In / done / 2026-10-13T11:14:23Z
- Merge-Bouncer / stuck / 2026-10-13T11:14:49Z
- Merge-Burler-Review / stuck / 2026-10-13T11:16:53Z
- Merge-Burler-Fix / stuck / 2026-10-13T11:19:43Z
- Merge-Bouncer / done / 2026-10-13T11:20:10Z
