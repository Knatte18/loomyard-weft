---
round: 2
focus:
  - "internal/gatecli/testmap.go `mapPackages` and internal/gatecli/testmappackage.go `mapPackage`: does a package skipped on the before-build dirty linked-file read leave the --max count untouched, while the after-tests skip still counts, and do the gatecli doc.go and the testMapCmd Long text say so?"
  - "internal/gatecli/testmap_integration_test.go `TestGateTestMapLinkedPackages` and `TestGateTestMap`: does a test fail if the HEAD-moved check over linked packages' files, the failed-build map's format marker with Commit and BuiltAt, or the before-build dirty read being fresh were broken?"
  - "internal/gatecli/testmaptree.go `nestedPackageDirs`: are the nested go.mod, testdata, and dot or underscore directory branches each pinned by a test row, and does its doc comment state the build-constraint bound?"
---
