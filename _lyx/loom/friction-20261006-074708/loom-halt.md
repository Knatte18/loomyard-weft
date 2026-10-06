loom halted blocked at Webster

```
batch 04-drop-per-skill-load: recover-batch refused with batch_failed (delete-not-done: SkillLoader.SkillLoadSequence, SkillLoader.SkillUnknown, Runner.LoadSkill, Runner.SkillUnknown and the claudeengine equivalents still resolve); card 4 deletes code that card 3 (orch reload) still uses, and the batch order runs 04 before 03
```

producer: Webster
state: blocked
anomaly: escalation-to-human
history_entries: 12
trace_file: /home/knatte/Code/loomyard-LYXHUB/skills-in-one-turn/.lyx/logs/trace-20261006T073807Z-2f71272978454dae-4163501.log

history:
- Preflight / done / 2026-10-06T07:18:29Z
- Loom-Preflight / done / 2026-10-06T07:18:36Z
- Discussion-Write / done / 2026-10-06T07:22:53Z
- Discussion-Bouncer / stuck / 2026-10-06T07:23:30Z
- Discussion-Burler / stuck / 2026-10-06T07:25:45Z
- Discussion-Bouncer / done / 2026-10-06T07:26:16Z
- Plan-Write / done / 2026-10-06T07:32:18Z
- Plan-Bouncer / stuck / 2026-10-06T07:33:05Z
- Plan-Burler / stuck / 2026-10-06T07:37:19Z
- Plan-Bouncer / done / 2026-10-06T07:37:55Z
- Batchifier / done / 2026-10-06T07:38:01Z
- Webster / stuck / 2026-10-06T07:46:36Z
