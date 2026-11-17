**Findings**

1. LOW. `internal/proc/memorycap.go:11`, the reworded doc line.
   - The line is over the 120-character prose limit (about 123 characters after `// `).
   - It also has a comma before "and" that opens a new clause; split after "too,".

2. LOW. `internal/proc/memorycap_linux.go:1-2`, the file header.
   - It says "It ends what the command leaves in the scope once the kernel kills it at the cap."
   - The fix ends the scope's leftovers after any OOM kill in the scope, including a kill of a child while the command lives.
   - `internal/proc/memorycap_linux_test.go:1-4` likewise says only "A kill at the cap ends what the child left in its scope".

No other twin left unchanged.

**Tool calls:** none. The context already held every changed file.
