# Card 3 — reconcile-hints

**Edit:**
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `internal/reedcli/up.go`

**Prosa:**
- `docs/overview.md`

**Intent:**
Rewrite every hint telling an existing hub to run `lyx config reconcile` to adopt a new key so it says the key takes its template default until reconcile writes it (D8); since card 2 the old wording sends operators on the sweep D1 removes.

- Both reed templates: `collapsed_rows`' comment says an already-materialized `reed.yaml` lacking the key takes this default until `lyx config reconcile` writes it (its retired `collapsed_strip_rows` clause stays);
  `debug_log`'s comment drops "existing hubs must run lyx config reconcile to adopt this key" for the same default-until-reconcile wording.
- `internal/reedcli/up.go`: the `up` help text's `debug_log` paragraph says the same instead of "existing hubs need lyx config reconcile after upgrading to adopt the key".
- `docs/overview.md`: the landing sentence saying an existing hub picks up `describe`, `describe_timeout_min` and `co_authored_by` with `lyx config reconcile --apply` says they take their template defaults until reconcile writes them.

Before finishing, grep `config reconcile` across `internal/`, `docs/` and `contracts/` and rewrite any other adopt-a-new-key hint the same way.
Leave the hints for what reconcile still does: materializing an absent file (`Load`'s absent-file error, the seed-only `models`/`burler` templates and `burlerengine`'s no-fans error), pruning removed keys, and describing the verb itself.

**ImpactSummary:** Comment and help-text wording only, in the reed templates, the `reed up` help and one overview sentence; no behavior change.
