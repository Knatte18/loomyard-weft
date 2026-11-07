```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the `_lym/landing/summary.md` edits (the resolve-error design bullet and the Tests bullets) for agreement with their surroundings.

- The reworded bullet matches `ErrUnresolved` in `internal/hubregistry/hubregistry.go:25`, which is documented as "names no hub or more than one hub", and a record that can't be read "does not match it". This matches `markUnresolvable` in `internal/configreg/configreg.go`.
- The "What changes" bullet (summary.md line 23) says an entry that resolves to none is dropped with a line. This still agrees with the new bullet, since it concerns only the refused cases.
- The "Display" bullet already says `read --envelope` adds `sending_repository`. The new Tests bullet only adds the present and absent assertions.
- The `hubs` bullet matches the `cli_test.go:468` subtest name. That name covers a listed shortname, a hub with no repository and a forged name.
- No twin was left stale.

Calls made:
1. `grep ErrUnresolved` in `internal/hubregistry/*.go`: to confirm the doc of `ErrUnresolved` matches the wording I checked in the new bullet. My context had it only through the review's description.
