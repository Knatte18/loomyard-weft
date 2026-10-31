# Card 4 — retired-key-way-forward

**Edit:**
- `internal/configengine#Spec`
- `internal/configengine#ValidateLocal`
- `internal/configengine_test#TestValidateLocal`
- `internal/configengine/doc.go`

**Intent:** Give a retired config key a way-forward text its local-layer refusal carries (D13), before any module retires a key with one.

`configengine.Spec` gains `RetiredWayForward map[string]string`, keyed by the same dotted keys as `RetiredKeys`: the text telling the operator where the key lives now.
`ValidateLocal`'s refusal of a unit that a retired key of a category other than `CategoryNone` covers, `key %q is %s and cannot be set in a local override`, appends `; way forward: <text>` when `RetiredWayForward` holds text for that covering key; `Layers` already wraps the refusal with the local file's path, so it names the file, the key and the way forward. A retired key with no text keeps today's message, and nothing else in the local or machine rule changes.

Test rows in `TestValidateLocal`: a spec whose retired key has category guard and a way-forward text refuses a local document setting it with a message ending in that text; the same key with no text keeps the old message.
The configengine doc's local-rule paragraph names the field.

**ImpactSummary:** One Spec field and one refusal gain an optional way-forward suffix; no module sets the field yet, so no existing refusal text changes.
