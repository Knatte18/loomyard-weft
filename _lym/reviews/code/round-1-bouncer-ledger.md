---
round: 1
ledger:
    - key: production-comments-bare-keys
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: test-file-comments-unswept
      rounds:
        - 1
      status: open
      class: scope
      severity: MEDIUM
      fix: design-wrong
    - key: keyname-guard-detector-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: retired-entry-field-renamed-as-config-key
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: reinterpret-split-comments-lost-words
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: keyname-guard-colon-skip-too-wide
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: design-wrong
    - key: keyname-guard-allowlist-per-file
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: change-description-overstates-coverage
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: keyname-guard-unset-prefix-missing
      rounds:
        - 1
      status: open
      class: design
      severity: NIT
      fix: mechanical
---
Round 1 opens the ledger; every key is new.

The findings fall into three groups:

- **Sweep coverage.** The by-name search missed phrasings in production comments, and `_test.go` comments were not swept.
- **The new guard in `internal/configreg/keynames_enforcement_test.go`.** Its detector is untested, its colon skip and per-file allowlist let bare keys pass, and `--unset "` is missing from its prefixes.
- **Accuracy of changed text.** A retired board-entry field reads as a live config key, two split comments in `reinterpret.go` lost words, and the change description claims more than the diff bears out.
