# Card 8 — burler-profile-key

**Edit:**
- `internal/shedrecipe#burlerRoundProfile`
- `internal/shedrecipe#burlerFixProfile`
- `internal/shedrecipe#TestBurlerHalfEntries_ColorSegment`
- `internal/burlercli#profileYAML`
- `internal/burlercli#decodeProfile`
- `internal/burlercli#burlerCLI.runCmd`
- `internal/burlercli#TestDecodeProfile`
- `docs/overview.md`

**Uses:**
- `internal/burlerengine#Profile`

**Intent:**
The recipe profile key `language-skills` and `lyx burler run`'s profile YAML key of the same name set `burlerengine.Profile.LanguageSkills` (D5).

`burlerRoundProfile` and `burlerFixProfile` each read `configBool(cfg, "language-skills", false)`, add `"language-skills"` to their `configRejectUnknown` key list, and set `LanguageSkills` on the profile they return.
`configBool` already refuses a non-boolean value with an error naming the key, so a recipe row with `language-skills: "yes"` fails construction naming `language-skills`.
The fix-row doc comment on `burlerFixProfile` names its third recognised key.

`profileYAML` gains `LanguageSkills yaml.Node` under the tag `language-skills`, and `decodeProfile` maps it: an absent key is false, a `!!bool` scalar decodes to its value, and any other node fails with `burler: profile YAML: key "language-skills" must be a boolean`, since `yaml.v3`'s own decode error names a line and a type but not the key.
`runCmd`'s `Long` example profile gains the line `language-skills: false  # true loads the hub's language skills after burler's own on both seats`.

`TestBurlerHalfEntries_ColorSegment`'s per-half loop, which already captures both halves' profiles through `captureReview` and `captureFix`, gains two subtests per half: `<half>LanguageSkills` sets the profile map's `language-skills` to `true` and asserts `profile.LanguageSkills` is true, and `<half>LanguageSkillsNotBool` sets it to the string `"yes"` and asserts the entry fails with an error containing `language-skills`.
Its doc comment says it covers the optional per-half keys that map onto the profile, `color_segment` and `language-skills`.
An absent key maps through `configBool`'s existing absent branch to false, which every existing row with no key already reaches.
`TestDecodeProfile` gains three rows: `language-skills: true` decodes to `LanguageSkills` true, an absent key to false (the zero `yaml.Node` branch `decodeProfile` adds), and `language-skills: "yes"` fails with an error containing `language-skills`.

`docs/overview.md`'s burler entry adds `language-skills` to its profile-driven list: a code round's seats load the hub's `language.yaml` skills after their own.

**ImpactSummary:** one new optional key in the two Burler profile mappers and the `lyx burler run` profile decoder; every existing recipe row and profile file omits it and maps to false, so nothing changes until card 9 sets it.
