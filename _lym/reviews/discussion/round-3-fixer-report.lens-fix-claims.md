```yaml
lens: fix-claims
outcome: reported
```

Task: check every claim the fix added or rewrote against the code (fix-claims lens).

- LOW — Constraints, new bullet: "A `-c` key enters the session-flags layer the effective config merges (`build_cli_overrides_layer` in `codex-rs/config/src/loader/mod.rs`, same tag)".
  Summary: the citation points at the wrong file for this function.
  Evidence: at `rust-v0.164.0`, `build_cli_overrides_layer` is defined in `codex-rs/config/src/overrides.rs:9`. `loader/mod.rs` only imports it (`use crate::overrides::build_cli_overrides_layer;`, line 35) and pushes the result as the `ConfigLayerSource::SessionFlags` layer. The claim itself holds. Only the location is wrong, so a reader who searches `loader/mod.rs` for the definition won't find it. A correct wording cites `codex-rs/config/src/overrides.rs` for the builder and `loader/mod.rs` for the `SessionFlags` layer push.

The other claims held against the Codex source already fetched into `.scratch/`:
- D6's statement that the hooks are replaced by the main checkout's when the worktree's root holds a `.codex/` directory. The layer is skipped when that directory is missing, and a missing `config.toml` still sets `load_root_checkout_hooks: true`.
- The `config_folder` rules claim.
- That `-s` returns `Legacy` syntax (`resolve_permission_config_syntax`).
- The seeding of workspace roots when no profile is selected.
- D5's "whether or not the worktree holds its own".

Tool calls:
- One `gh api` fetch of `codex-rs/config/src/overrides.rs` plus a grep of the loader. My context showed the import line but not where the function is defined.
