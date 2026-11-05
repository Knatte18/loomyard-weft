```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I walked each function and branch the diff adds or changes with concrete inputs. I ran no code and read yaml.v3 source once.

## List: function or branch, case run, verdict

- `FollowTemplate`, base and template both empty or comment-only: base root nil, the file gains every template key. Correct.
- `FollowTemplate`, empty or comment-only file: a synthesized root is appended and everything is added. Correct.
- `FollowTemplate`, unchanged file: the input bytes come back unchanged, because `Differs` alone does not set `Changed()`. Correct.
- `FollowTemplate`, non-mapping top level in base, template or file: error naming which document. Correct. The caller misnames the file; see F1.
- `follow`, existing and template are mappings but the base is a scalar: compared whole, so the operator value is kept, and `Differs` is reported if the template differs. Correct.
- `follow`, a removal empties the mapping: the mapping is removed, and the next pass reads that as an operator removal (base `{}`, existing absent), so it stays gone with no report. Correct.
- `followMapping`, key only in the base, absent from file and template: existing is nil and not equal to the base, so it is kept and no report is made. Correct.
- `followMapping`, `templateKey` is dereferenced in the append branch: that branch is reachable only through `Added`, which requires a non-nil template. No nil dereference. Correct.
- `followUnit` with a base, record rule 2:
  - hub equals base and template moved: `Updated`. Correct.
  - hub differs from base: kept, `Differs` when the template is a third value. Correct.
  - key the template added that the hub lacks (base also lacks it): `Added`. Correct.
  - hub lacks, base has, template equals base: silent, stays removed. Correct.
  - key the template removed: removed when hub equals base, otherwise `Differs` with "template absent". Correct.
- `followUnit` without a base: lacking key added, differing value kept and listed, file-only key kept silently. Correct. This is rule 3's fallback for a hub with no base.
- `ReconcileHubWideAt` seed-only branch, absent file and no prime copy: template verbatim, break before the follow, base written on apply. Correct.
- `ReconcileHubWideAt` seed-only branch, empty present file: `os.ReadFile` returns a non-nil empty slice, so it migrates, then `FollowTemplate` adds all keys. Correct.
- `ReconcileHubWideAt` seed-only branch, absent file seeded from a prime copy: `haveBase` is false, so the prime copy gains the missing template keys. Correct.
- `readTemplateBase`: `os.IsNotExist` gives present=false. A read failure returns a `*FileError` naming the base path. Correct.
- Base write on apply: `MkdirAll`, then `AtomicWriteBytes`, written only when absent or stale. A dry run writes nothing. Correct.
- Base write ordering: the file is written first, then the base. If the base write fails after the file write, the stale base with the new file produces no change and no report on the next pass. Correct.
- `ChangedFilesRel`: includes the base even when `Applied` is false, and keeps the legacy files. Correct for the configcli, fabric reconcile and walk callers.
- Walk failure path: `snapshotConfig` covers the base, and `restoreConfig` removes a base that was absent before. Correct.
- `clone.go`: records the base as a written file, and the whole-repo `b.Commit` carries it. Correct.
- `followedLines` and `notifyTemplateFollowed`: `followed` is reset on each `Write` call, and mail goes out only after the commit, skipping on `res.Skipped`. Correct.
- `UnsettledFormerBlocks`: a pair differing under `light` is reported, and an absent block lists nothing. Correct.
- `template.yaml`: the three-line grep of the diff shows `light` loses the key in all four recipe blocks, and `docs` and `speed` are untouched. Correct.

## Findings

### F1: MEDIUM, design
- **Location:** `internal/configsync/configsync.go:1044-1049`, `internal/yamlengine/follow.go:43-46`
- **Summary:** An unparseable or non-mapping template base makes `FollowTemplate` fail with `parse base YAML`, and the caller attributes it to the module file, not the base.
- **Effect:** The `FileError` at line 1117 names `cfgPath`, so the operator is pointed at `profiles.yaml` and the base file is never named. The error carries no way forward such as deleting the base so the next reconcile treats the file as having no base.
- **Walk consequence:** The failure aborts the whole hub walk, and the stamp stays stale. Every start verb then retries and fails the same way.
- **Evidence:** `follow.go:43` returns the `base` parse error. `configsync.go:1046` passes `templateBase` into the same call that parses the module file. `configsync.go:1117` wraps the error with `cfgPath`.

### F2: LOW, design
- **Location:** `internal/yamlengine/follow.go:75-84`
- **Summary:** A changed file is re-encoded with a default `yaml.v3` encoder, which wraps flow mappings once a line passes column 80, and which I expect drops blank lines between entries.
- **Effect:** `profiles.yaml` holds long flow mappings, such as the `light` tapestry loom line of about 170 characters. After a follow they are folded across lines. The values are the same, but the operator's file layout changes. The doc comment promises each node's style is kept.
- **Evidence:** `emitterc.go:331-335` in yaml.v3 v3.0.1 turns the default width 0 into 80, and `emitterc.go:673` and `emitterc.go:697` break flow items past that width. `follow.go` sets only indent, never width.
- **Not run:** I did not run an encode, so the exact output is unconfirmed.
- **Tests:** The tests check only substrings shorter than 80 columns (the integration test's `"loom: {plan_max_bounces: 7, discussion_max_bounces: 2,"` prefix and the short `lightDarn` blocks), so they would not catch the wrapping.

## Tool calls
1. `Read internal/yamlengine/layers.go:600-670`: `parseMappingRoot`, `nodesEqual` and `mappingIndex` were not in my context.
2. `Bash` grep of `best_width` in the yaml.v3 module cache: to check whether the encoder wraps long flow mappings by default.
