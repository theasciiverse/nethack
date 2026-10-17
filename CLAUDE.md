# NetHack (Asciiverse packaging)

NetHack 5.0 by the NetHack DevTeam, packaged for the Asciiverse catalog and
as system packages. Not our code: keep changes small, guard them with
`AV_PATHS`, and list every one in PATCHES.md.

- Remotes: `upstream` is github.com/NetHack/NetHack, `origin` is
  github.com/theasciiverse/nethack.
- Branch `asciiverse` = tag `NetHack-5.0.0_Released` plus our commits.
- Release: tag `v5.0.0-av.<n>` and push it.
- Local static build: `docker run --rm -v "$PWD":/src -w /src alpine:3.22
  sh asciiverse/build.sh`; the install lands in `out/` (owned by root, so
  chown it afterwards). Generated Makefiles and objects are gitignored.
- Data beside the binary (else `/usr/share/nethack`), player files in
  `~/.local/share/nethack`: `asciiverse/avpaths.c`.
- Not the same title as `hack` (Hack 1.0.3 of 1985, NetBSD's, in netbsd-games),
  NetHack's ancestor.
- Preview: `python3 asciiverse/record-preview.py` after a build (needs pyte).
