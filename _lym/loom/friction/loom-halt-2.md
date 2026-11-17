loom halted blocked at Plan-Write

```
gate failed its first check and the session was never re-prompted, since its re-prompt could not be sent: shuttle: strand "7607bc3bbc115195586bf06a8f38d487" is not tracked by reed — either its run completed and was cleaned up, or reed's strand table was reset under it (a reed remove/down, or a lost or rebuilt reed.json); check "lym reed status"; findings: /home/knatte/Code/loomyard-LYMHUB/launch-manifest-parity/.lym/shed/launch-manifest-parity/gate-findings/Plan-Write.md
```

producer: Plan-Write
state: blocked
anomaly: escalation-to-human
history_entries: 6
trace_file: /home/knatte/Code/loomyard-LYMHUB/launch-manifest-parity/.lym/logs/trace-20261117T144036Z-54ce7b64b62a3654-2730623.log

history:
- Preflight / done / 2026-11-17T14:20:19Z
- Motif / skipped / 2026-11-17T14:20:24Z
- Drift-Baseline / skipped / 2026-11-17T14:20:28Z
- Parts-Write / skipped / 2026-11-17T14:20:32Z
- Parts-Plan-Write / skipped / 2026-11-17T14:20:35Z
- Plan-Write / stuck / 2026-11-17T14:43:16Z
