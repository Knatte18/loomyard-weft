# Spawned code roles load the hub language skills

`role-language-skills` · Task · enhancement, config, shuttle · running · Publish

## Seen (operator, 2026-10-14)

The darn writer of `delete-order-own-edit` writes Go but loaded only `scribe:prose` and `scribe:code-quality`.
No spawned code-writing or code-reviewing role loads a language skill:

- darn writer: `darnSkills` in `internal/darnengine/spec.go`, `scribe:prose`, `scribe:code-quality`; no `scribe:testing` either.
- Webster master and recovery: `roleSkills` in `internal/websterengine/roles.go`, `scribe:prose`, `scribe:code-quality`, `scribe:testing`.
- Burler seats: `burlerSkills` in `internal/burlerengine/engine.go`, the same three.
- Merge conflict resolution: `conflictSkills` in `internal/mergeresolve/spec.go`, `scribe:prose`, `scribe:code-quality`.

`scribe:golang-comments`, `scribe:golang-testing` and `scribe:golang-build` exist, and `pattern/PATTERN-test-economy.md` cites `scribe:golang-testing` as holding the general rules, yet no role gets them.

## Idea

- The hub's config names its language skills (e.g. Go: `scribe:golang-comments`, `scribe:golang-testing`, `scribe:golang-build`; a C# hub its `csharp-*` ones), since lyx serves hubs in other languages and must not hard-code Go.
- Each code-writing or code-reviewing role appends the hub's language skills to its own list; which roles get which (comments for writers and reviewers, testing for test writers, build for whoever runs the build) is to be decided.
- The darn writer also gains `scribe:testing`, as Webster and Burler have.
