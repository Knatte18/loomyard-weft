# lym config drift refuses the whole-map capture and compare form its own key description prescribes

`drift-config-whole-map-set` · Task · bug, drift, config

From [issue #19](https://github.com/Knatte18/loomyard/issues/19).

## Seen

`lym config drift --set 'capture={command: "...", dir: "...", timeout_min: 30}'` is refused with "unknown config key(s): capture, compare", although the key's own description in `--resolved` prescribes that whole-map form.
Setting `capture.command`, `capture.dir` and `capture.timeout_min` one by one works.

## Expected

A whole-map `--set` of `capture` or `compare` is accepted as the key description says, or the description names the per-key form instead.
