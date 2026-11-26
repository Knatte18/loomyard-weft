---
round: 1
entries:
  - key: shellguard-doc-describes-claudeengine-golden
    status: closed
    finding: F1
    reason: "4a83092c6 deletes the golden paragraph and its blank comment line from internal/shellguard/doc.go; the command remains in the updateSettingsGolden doc comment in claudeengine/settings_test.go"
  - key: settings-golden-path-literal-repeated
    status: closed
    finding: F2
    reason: "fcbccde0d declares settingsGoldenPath once before updateSettingsGolden; the -update usage string, the golden write and the golden read all use it"
---
