loom halted blocked at Webster

```
Batch 07-report-config: recover-batch 07 refused with batch_failed (create-not-done/7-report-config[blocking]: Create target "internal/reportengine/template.yaml#" and the plan:internal/reportengine#Config, #ConfigTemplate, #ConfigSpec, #LoadConfig and #TestLoadConfig targets still do not resolve, since the config module was built in internal/reportconfig to avoid an import cycle; way forward: lym webster recover-batch 07)
```

producer: Webster
state: blocked
anomaly: escalation-to-human
history_entries: 26
trace_file: /home/knatte/Code/loomyard-LYMHUB/raddle-recipe/.lym/logs/trace-20261105T104035Z-f4c942698e923c77-2682084.log

history:
- Preflight / done / 2026-11-05T08:33:03Z
- Motif / done / 2026-11-05T08:33:06Z
- Discussion-Write / done / 2026-11-05T08:46:04Z
- Discussion-Bouncer / stuck / 2026-11-05T08:46:33Z
- Discussion-Burler-Review / stuck / 2026-11-05T08:53:31Z
- Discussion-Burler-Fix / stuck / 2026-11-05T09:00:17Z
- Discussion-Bouncer / stuck / 2026-11-05T09:00:55Z
- Discussion-Burler-Review / stuck / 2026-11-05T09:08:38Z
- Discussion-Burler-Fix / stuck / 2026-11-05T09:16:02Z
- Discussion-Bouncer / stuck / 2026-11-05T09:16:46Z
- Discussion-Burler-Review / stuck / 2026-11-05T09:24:24Z
- Discussion-Burler-Fix / stuck / 2026-11-05T09:29:04Z
- Discussion-Bouncer / done / 2026-11-05T09:29:44Z
- Drift-Baseline / skipped / 2026-11-05T09:29:46Z
- Parts-Write / skipped / 2026-11-05T09:29:48Z
- Parts-Plan-Write / skipped / 2026-11-05T09:29:50Z
- Plan-Write / done / 2026-11-05T10:07:11Z
- Plan-Bouncer / stuck / 2026-11-05T10:07:37Z
- Plan-Burler-Review / stuck / 2026-11-05T10:18:58Z
- Plan-Burler-Fix / stuck / 2026-11-05T10:25:09Z
- Plan-Bouncer / stuck / 2026-11-05T10:25:42Z
- Plan-Burler-Review / stuck / 2026-11-05T10:35:57Z
- Plan-Burler-Fix / stuck / 2026-11-05T10:40:03Z
- Plan-Bouncer / done / 2026-11-05T10:40:31Z
- Drift-Await / skipped / 2026-11-05T10:40:35Z
- Webster / stuck / 2026-11-05T11:27:18Z
