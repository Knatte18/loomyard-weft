# Driver repair records: Webster-Burler steps killed by SIGQUIT

```yaml
run_id: card-batching
failure: a `lyx shed step` job died on SIGQUIT with no envelope while parked in shuttle's Wait after the Burler session had stopped and written its output files
interrupt_policy: reinvoke
```

## Round 1

- Interrupted trace: `83efe16ee7c6dc40` (`burler: round starting` round=1, then `shuttle: run started`; no `shed: step done`).
- Status read: `current_producer` Webster-Burler, `history_length` 25, state running, so unchanged.
- Action: re-invoked `lyx shed step card-batching` (repair 1 of 2 for key `(Webster-Burler, 25)`).
- Outcome: trace `c2275122460eeb3d` logged `burler round producer reached with the highest complete round unjudged; handing back for judgment instead of running a fresh round`; the run advanced to Webster-Bouncer, history_length 26.

## Round 2

- Interrupted trace: `6fdd210925eac3c4` (`burler: round starting` round=2, then `shuttle: run started`; no `shed: step done`).
- Status read: `current_producer` Webster-Burler, `history_length` 27, state running, so unchanged.
- Action: re-invoked `lyx shed step card-batching` (repair 1 of 2 for key `(Webster-Burler, 27)`).
- Outcome: trace `dde36fd876591cad` logged the same unjudged-round handback; the run advanced to Webster-Bouncer, history_length 28.
