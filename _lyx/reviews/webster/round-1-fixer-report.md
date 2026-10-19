## Changes

- F1, fixed as proposed (`e379a702f`):
  `internal/fabricengine/destroy_test.go` drops `removePortal` from the idempotent-teardown list,
  and `internal/fabricengine/destroy_containment_toctou_integration_test.go` now says "launcher teardown".
- F2, fixed as proposed (`268316623`):
  `PATTERN.md` restores main's wording for `PATTERN-push-both-sides`, `PATTERN-recipe-setting-lookup` and `PATTERN-batcher-registry`.
  Only the two declared containment entries differ from the task's base now.
  `PATTERN-shed-recipe-registry` differs from main's tip because main moved on, not because of this task, so it was left alone.
- Self-check follow-up (`6ffd4a27e`):
  `internal/fabricengine/ancestors_test.go` `TestRefuseUncontainedPath` comment and `//testtiming:keep` text say "launcher teardown helpers" instead of "portal and launcher teardown helpers".

- Gate finding, `PATTERN.md` over its headroom line by 48 bytes:
  `PATTERN-hub-containment` returns to main's wording, since its background file already states that the hub workspace file is hub-only;
  `PATTERN-fabric-write-containment` now reads "Writing `_launchers` or the workspace file from `fabricengine`: via an `os.Root` at the hub, never raw `os.MkdirAll`/`os.WriteFile`/`fslink`."
  `PATTERN.md` is 15562 bytes, under the 15564-byte line.

- Gate finding, fixed-column-wrapped comment at `destroy_test.go:501`:
  the F1 comment is rebroken at its colon, so each line is a clause rather than a column-wrapped fragment.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: LOW, a missed twin of F1 in `ancestors_test.go:170,173`. Fixed as proposed (`6ffd4a27e`).

Two remaining hits of the older wording were judged outside F1's premise and left as they are:
`livestate_doc_test.go:137` narrates a past run, and `lyxcwd/enforcement_test.go:233` explains a still-valid `_portals` allowlist entry.

## Disputed

None.

## Deferred

None.
