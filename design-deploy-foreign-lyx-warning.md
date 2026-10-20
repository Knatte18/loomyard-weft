# update-plugins warns about /usr/bin/lyx, which is the LyX document processor, not a stray lyx

`deploy-foreign-lyx-warning` · Note · bug, deploy

Imported from [issue #766](https://github.com/Knatte18/loomyard/issues/766).

Seen 2026-10-20 on OSL-1046 (Ubuntu 24.04): update-plugins.sh printed 'WARNING: another lyx on PATH: /usr/bin/lyx - remove it' and the same for /bin/lyx. /usr/bin/lyx is root-owned, from 2024, and is the Ubuntu LyX document processor package; /bin is a symlink to /usr/bin, so it is one file named twice. The production lyx in ~/go/bin comes first on PATH, so nothing is wrong. Workaround: ignore the warning. Expected: the check recognises a foreign binary of the same name (for instance by build info or a version probe) and does not ask to remove it, and it does not report one file twice through a symlinked directory.