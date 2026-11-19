```yaml
lens: fix-affected-set
outcome: reported
```

Findings (lens fix-affected-set):

**Unchecked:** F2, F4–F9. F4–F9 name a single test row or file, and F2 sits entirely in one file. None has a set premise.

1. **LOW**, F3: `internal/landingshed/doc.go:665-672`, the `DraftBeamDescription` paragraph.
   - **Summary:** the package doc still lists the draft as the entry, the commits and the `## Changes` list, and omits the Merge danger placeholder section.
   - **Evidence:** `beamdescribe.go:95` says the draft ends with a Merge danger section whose door and blast radius are placeholders. `beamcli/doc.go:142` was updated to say so. This doc is the third description of the draft and was left as it was.
   - **Reach:** line 673 says `RecordBeamDescription` validates "with the check the Describe gate runs". That does not say an unedited draft is refused, and the review's F3 set (help, package doc, way_forward) did not name this file.

2. **No finding**, F1: the only code that scans a body for the `## Merge danger` heading or for section ends is `mergeDangerSection` and `isSectionHeading`, both in `description.go`. The fence handling covers every member of that set.

3. **No finding**, F3's other sites: the stencils and specs that mention the section (`darn-template-write.md:73`, `landing-template-describe.md:56`, `role-beams.md:49`, `loom-rubric-code-review.md:99`) describe the final section the writer calls. None describes the draft's contents.

4. **No finding**, F10: `loom-template-discussion.md` is the only file that carries `{{.glossary}}`.

**Tool calls:**
1. `grep -rn "Merge danger"` over `.go` and `.md` files, excluding tests: my context lacked the set of sites that name the section.
2. `grep -rln` for `describe --draft`, `DraftBeamDescription` and `beamMergeDangerDraft`: my context lacked the set of files that describe the draft.
3. `grep -rn` for `isSectionHeading`, `mergeDangerSection` and `MergeDangerHeading`, plus `.glossary`: my context lacked the callers of the changed scan and the files that use the glossary marker.
4. `grep -n -i "draft"` in `landingshed/doc.go` and `beamcli/deps.go`: my context lacked their draft wording.
5. `Read` of `landingshed/doc.go:662-677`: my context lacked the draft paragraph's text.
