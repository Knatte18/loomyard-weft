# lyx config --set writes a per-round model list as a string

`config-set-round-list` · Task · bug, config · running · Describe

## Seen (2026-10-14)

`lyx config loom --set 'plan_review=[opus[medium], sonnet[high]]'` wrote `plan_review: "[opus[medium], sonnet[high]]"`, a quoted string, not a list.
The JSON form `'plan_review=["opus[medium]","sonnet[high]"]'` was also written as a string.
`plan_review` is one model-spec or a per-round list, and the template's default is a quoted string, so `--set` keeps the string type; the "a list-valued key takes a YAML flow list" path in `lyx config --help` does not reach a key that takes either shape.
The same holds for every per-round model key: `review`, `fix`, `fan_review` and each segment's `*_review` and `*_fix`.
The orch then had to revert both layers by hand; a string value of that shape is not a valid model-spec.

## Wanted (operator, 2026-10-14)

`--set` must support setting a per-round model list on these keys, so a per-round model change needs no editor session.
A value it cannot write in the key's shape is refused, never written as a string.
