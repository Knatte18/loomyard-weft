# batcher: a hub profiles map hides the template's profiles instead of adding to them

`batcher-hub-profiles-add` · Note · bug, config, webster

Imported from [issue #834](https://github.com/Knatte18/loomyard/issues/834).

Seen (2026-10-22): the hub's batcher.yaml added one profile, `wide`; `lyx config batcher --resolved` then listed only `wide`, and the template's `cautious` and `identity` no longer resolved. Workaround: copy `cautious` and `identity` into the hub layer with `--set 'profiles.<name>={...}'`, which pins their values there. Expected: a hub layer's profiles add to or override the template's by name, as the reconcile help's per-name map wording suggests.