# Config keys nest under the area they govern, named with dots

`config-keys-nested-by-area` · Task · sketch, config, loom, drift, docs · done

Decision record: [Sketch](decision-config-keys-nested-by-area.md)

## Seen

`loom.yaml` holds its keys flat with the area spelled into each name: 19 start `discussion_`, 13 `code_`, 8 `split_`, 3 `pr_`, and more.
Elsewhere a key is named bare without its file, such as `judge_ask_parent` in `drift.yaml`, which reads like the Bouncer's judge of a perch.
T11's change description and halt notices name keys this way, so a reader cannot tell which area a key governs.

## Wanted

Operator, 2026-11-23: a key's name shows the area it sits in, with a dot per level, as `drift.judge_ask_parent` and `loom.discussion.review_ask_parent`.

- `loom.yaml`'s prefixed keys become maps per area: `discussion`, `code`, `split`, `pr` and the others, such as `discussion.review_ask_parent` and `code.max_bounces`.
- `run_settings`, profiles, `lym config --set`, every template comment, doc, stencil and halt notice use the dotted full name, file included.
- Existing hub layers and board `run_settings` are carried over by a reinterpretation, as other key moves were.
- Decide which other templates have flat prefixed keys that belong in the same change.
