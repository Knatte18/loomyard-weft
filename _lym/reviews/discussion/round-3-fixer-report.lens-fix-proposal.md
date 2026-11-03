```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the review proposals the fixer carried out (F1's variant, and F2 to F5).

Proposals challenged: F1 (fixed differently: the Warn applies only when the worktree root holds a `.codex/` directory), F2, F3, F4, F5.

1. **LOW: F2, the new Constraints bullet** ("…when no permission profile is selected… which `-s` sets…; lym's home `config.toml` selects no profile").
   The proposal treats `-s workspace-write` as enough to reach the writable-roots seeding. It isn't.
   - In `codex-rs/core/src/config/mod.rs`, around lines 3691–3698, seeding needs `selected_profile_id.is_none()` as well as legacy syntax. `-s` settles only the syntax (`resolve_permission_config_syntax` returns `Legacy`).
   - The profile is chosen by `resolve_effective_permission_selection` → `resolve_default_permissions` (lines 5026 and 5044–5067). It reads `configured_default_profile_id` from the merged config, and a `requirements.toml` with `allowed_permission_profiles` forces a fallback profile.
   - The merged config also takes a system `config.toml` and a trusted project `.codex/config.toml`, so either could select a profile even though lym's home selects none.
   - Concrete case: a committed `.codex/config.toml` with `default_permissions = ":workspace"`, or a managed `requirements.toml`, selects a profile. Seeding is then skipped and the weft root is dropped with no error.
   - Fix: the clause should say that no layer, and no `requirements.toml`, selects a profile. Or Prepare's existing Warn on a committed `.codex/config.toml` could be named as the bound for the project case.

2. **LOW: F4, D6 Bound** ("…and other rounds' review files, tracked content the run's commits carry.").
   Dropping "and markers" fixes the tracked-content claim. But the Bound now names no `.lym` content the writable root reaches, and the new D1 sentence puts every marker under the anchor's `.lym` inside that root.
   - Concrete case: the assistant writes its own marker as `done` before its seat gate passes. The chair's `lym shuttle await <marker>` returns and the chair reads an ungated report.
   - D1 says "A done ending is written only after the assistant's seat gate has passed". That holds for the engine, but the session itself can still write the marker.
   - Before the fix the Bound at least listed markers as reachable. The general "anywhere in the weft checkout" line remains, but nothing names this effect.
   - Fix: add one clause saying that in `.lym` the root reaches the seats' markers and reports, untracked, and that only the prompt keeps the assistant off them.

3. **No finding: F1 variant.** It matches the source the fixer read:
   - `discover_project_layers` skips a directory with no `.codex/` folder.
   - `load_root_checkout_hooks: true` is set even when that folder holds no `config.toml`.
   - The condition assumes Codex starts in the worktree root, which D6's launch implies.

4. **No finding: F3.** A missing field and an empty one read the same way (no model choice, bare-round token), so this does not need the separate flag the fan record needs.

5. **No finding: F5.**

Tool calls:
- Grep of `.scratch/config-mod.rs.txt` for `configured_default_profile_id` and `implicit_default_permissions`: the context had the seeding condition but not where a profile gets selected.
- Grep and `sed` of the same file around lines 4990–5014: to see where `configured_default_profile_id` and `requirements_toml` feed the selection, which the context lacked.
