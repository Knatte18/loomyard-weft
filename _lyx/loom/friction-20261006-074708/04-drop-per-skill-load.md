# Card 4 ordered before card 3, but card 4 deletes what card 3 still uses

Card 4 says "cards 2 and 3 already moved every caller off" `Runner.LoadSkill` and `Runner.SkillUnknown`, but the batch order runs 04 before 03.
`internal/orchcli/session.go` (`runnerSession.LoadSkill`/`SkillUnknown`) and `internal/orchengine/watcher.go` (`Session.SkillUnknown`, `w.session.LoadSkill`) still call them, and card 3 alone owns those files.

Deleting the `Runner` methods breaks `go build ./...`, the card's own gate.
Its `Verify:` (`! grep -rn 'SkillUnknown\|SkillLoadSequence' --include=*.go internal`) also matches `internal/orchengine`, so it can only pass once card 3's `Session` seam change has landed.
Making card 4 pass would mean doing card 3's work inside card 4, so I stopped without editing anything.
The plan needs card 3 ordered before card 4 (card 4 depends on 3).
