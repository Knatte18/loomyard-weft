```yaml
lens: code-affected-set
outcome: reported
```

code-affected-set lens: what the diff's changes force outside the lines they touch.

**Per change, the set found and whether it is covered**

1. **Edit directive, Claude and Codex copies** (`contracts/stencils/engine/{claude,codex}/edit-directive.md`): three build-resolution lines are replaced by the ban.
   - Readers:
     - `editdirective.Directive` reads the copy by `EngineBlock` (`internal/editdirective/editdirective.go:18`). The text is opaque to it, so nothing changes.
     - `shuttleparity/prepare.go:27,44` matches the prompt against the engine's copy as read. It is not text-pinned, so it is covered.
     - The `shuttleengine/manifest.go:23` channel name is unchanged.
   - Tests: `TestDirective_ReadsTheEnginesCopyWithoutBanner` seeds from the shipped registry (`internal/testkit/stencilkit/stencilkit.go:36`), so it reads the new text. Covered.
   - Golden fixtures: no fixture under `internal`, `contracts` or `tools` holds the removed sentence or the "Scratch files go under" line. Nothing is stale.
   - Docs describing the directive:
     - `internal/editdirective/doc.go` is updated in the diff. Covered.
     - No doc under `docs` outside `docs/research`, `plugins` or `contracts/specs` describes the directive.
     - The copy in `docs/research/.../prompt.md:26` is a captured prompt from a past run, which is a measurement record. No finding.
   - The two copies stay in step: a diff of the Codex copy, with its tool wording swapped, against the Claude copy differs only in the tool-wording and banner lines.
   - Engine-parity and stencil-ownership backgrounds: neither says anything about scratch content. Nothing is forced.
2. **Orch role note** (`contracts/orchrole/orch-role.md:452-453`): no orchrole test asserts the Notes section. The ban agrees with the directive's wording. Covered.
3. **`internal/editdirective/doc.go`**: the package doc's "each copy carries every rule above" (doc.go:27-28) still holds, since both copies carry the ban. Covered.
4. **`fabricengine` doc** (`internal/fabricengine/doc.go:655-657`): it still states that a build under `.scratch` resolves a relative reference from the target in the weft. That is a fact about the link, not an instruction, and it agrees with the ban's "two paths" reason. It is not left wrong.
5. **Repository rules, PATTERN.md and `pattern/`**: the range has no diff under `PATTERN.md` or `pattern/`, and the directive declares no PATTERN change. `PATTERN-lym-files` names no scratch build rule. Nothing to land.
6. **Commit messages**: the range's two commits carry no `Pattern-Departure:` trailer, and neither needs one.
7. **Other shipped instructions that might contradict the ban**:
   - `contracts/stencils/retro/retro-template-task.md:23,26` runs `forkaudit.sh --work .scratch/forkaudit`. The script's header says `--work` holds intermediate files only, with no build, restore, compile or checkout. No contradiction.
   - `plugins/ly/skills/board-policy/SKILL.md:28`, `beam-template-board-tidy.md:27` and `orch-role.md:223,468` keep drafts in `.scratch/`. These are drafts, not builds.
   - `contracts/specs/loom-plan-spec.md:159` uses a `.scratch/` Create target as a transient draft, not a build.

**Findings**

None. Every member of the affected set is covered or unaffected.

**Tool calls**
1. Grepped the repository for leftover build-guidance text and the PATTERN scratch rules, because the manifest held no repository-wide search for the removed sentences.
2. Read the hubforge and gatecli hits, `stencilkit.Seed`, and the doc and pattern mentions of the edit directive, to rule out false hits and confirm the test reads the shipped copies, which the manifest lacked.
3. Grepped for golden fixtures with the directive text, orchrole tests, and the PATTERN diff stat, because the manifest held no fixture or orchrole-test search.
