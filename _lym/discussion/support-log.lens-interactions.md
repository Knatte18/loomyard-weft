```yaml
lens: interactions
outcome: reported
```

## Findings

1. **MEDIUM — D2 × D3 (stale record wins).** D3 step 1 says a record read under the lock "wins and no pick is made"; D2 says a record naming an account no longer listed is picked again. Taken literally, the under-lock re-check lets a stale record win, so the session launches with an unlisted account. Evidence: D3 step 1 has no "and its account is still listed" clause; the D2 acceptance criterion ("replaced by a new pick") contradicts D3's literal steps. Fix: step 1 treats a record whose account is not listed as absent.

2. **MEDIUM — D4 × D6 (env-module dir invisible to readers).** With no accounts configured, D4 keeps an `env` module `CLAUDE_CONFIG_DIR` in force for panes, but D6's fallback reads "`CLAUDE_CONFIG_DIR` when set" — lym's own process environment, which the `env` module never exports to lym's own readers (it reaches only panes, gate tiers and Drift steps). So readers fall back to `~/.claude` while sessions write under the env-module dir: transcripts, registry, resume and plugins lookup all miss. Same gap for D6's `lym report` set ("every configured account plus the default"). Evidence: exploration found `envconfig` reaches panes via `hubgeom.paneEnv`, gates and Drift only. Fix: D6's fallback reads the `env` module's `CLAUDE_CONFIG_DIR` value, and the report includes it.

3. **MEDIUM — D5 × D3 (stale overwrite).** Several sessions on one account each run the status line verb; an idle session's verb re-sends the `rate_limits` it got at its last API response. "Write only when changed" compares against the file, so an idle session's older (lower) figures overwrite a busier session's newer ones whenever they differ. D3's eligibility then reads an under-reported percentage. Evidence: D5 "only when they changed", no newer-wins rule. Fix: merge per window — later `resets_at` wins; equal `resets_at` keeps the higher `used_percentage`.

4. **LOW — D3 steps 2 and 4 (fallback can pick a logged-out account).** The soonest-reset fallback (step 4) runs when "no account is eligible", and eligibility (step 2) bundles the credentials check with headroom, so the fallback may take an account with no `.credentials.json`. Fix: the fallback ranges only over accounts whose credentials file exists.

5. **LOW — D1 × D5 × D4 (path comparison unnormalised).** D1 expands `~/`; D4 compares against the default `~/.claude`; D5 matches the session's `CLAUDE_CONFIG_DIR` string against `config_dir`. No clean/symlink-resolve rule is stated, so a trailing slash or symlinked home makes the verb write nothing and D4 may miss the default-directory case. Fix: state one normalisation for every `config_dir` comparison.

6. **LOW — D1 (empty list) × D2 (sticky record).** If the operator empties `claude.accounts`, a worktree whose record names a non-default account gets no export; its live sessions then resume under the default dir and lose their transcripts. The record is silent; state it as the operator's consequence, or keep the record's account while its directory exists.

## Pairs checked

- D1 × D4 (empty list, default dir unset): held.
- D1 × D5 (window thresholds vs recorded windows): held.
- D1 × D3 (credentials stat vs `config_dir`): held — default account's credentials are at `~/.claude/.credentials.json`.
- D2 × D3 (record re-read under lock): broken — finding 1.
- D2 × D6 (readers take recorded account): held; a re-pick after account removal loses old transcripts, which D2 logs.
- D2 × D7 (sticky prime/beam record): held.
- D2 × D8 (resume keeps account): held.
- D3 × D5 (eligibility reads usage files): broken under concurrent writers — finding 3.
- D3 internal (step 2 credentials vs step 4 fallback): broken — finding 4.
- D3 × D7 (prime/beam picks advance the shared counter): held.
- D4 × D5 (unset var maps to default dir in the verb): held.
- D4 × D6 (no-accounts fallback): broken — finding 2.
- D4 × D8 (resume command carries the export): held.
- D1 × D2 (emptied list vs sticky record): weak — finding 6.
- D1 × D4 × D5 (path comparison): weak — finding 5.

## Acceptance criteria and constraints vs decisions

- D2 re-pick criterion contradicts D3 step 1 (finding 1).
- D3 concurrent-sessions criterion: held.
- D3 "none having one, the launch fails": held, but fallback ordering ambiguous (finding 4).
- D5 verb-writes criteria don't cover the multi-session overwrite (finding 3).
- D6 report criterion misses the env-module dir (finding 2).
- D1 empty-list neither-sets-nor-unsets: held. D4 resume-command export: held. D8: held.
- `PATTERN-shuttle-provider-seam` and `PATTERN-lym-files` constraints: consistent with D4, D5, D2.

Further tool calls: none — my context held the exploration and both reviewed files.
