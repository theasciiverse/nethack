# Changes from upstream

This is the Asciiverse packaging of NetHack 5.0.0 by the NetHack DevTeam,
released 2026-05-02, under the NetHack General Public License
(`dat/license`, SPDX `NGPL`). The `asciiverse` branch starts at upstream's
tag `NetHack-5.0.0_Released` (commit `16ff591`) from
[NetHack/NetHack](https://github.com/NetHack/NetHack), the `upstream`
remote. Everything after it is ours.

## Where the files go

Upstream's Unix build expects one playground folder, made at install time
and writable by the game (setgid `games` on shared machines). Here the game
runs from a folder nobody needs to write to, and each player has their own:

- Read-only data (`nhdat`, `sysconf`, `license`, `symbols`): the folder the
  binary sits in when `nhdat` is there, which is how av installs it, else
  `/usr/share/nethack`, the compiled-in `HACKDIR`, where the system
  packages put it (`asciiverse/avpaths.c`, `av_hackdir()`; Linux reads
  `/proc/self/exe`, macOS `_NSGetExecutablePath`). The game changes to that
  folder as NetHack always does, and reads `sysconf` there (`SYSCF_FILE` is
  the relative `sysconf`).
- Everything written (record, logfile, xlogfile, livelog, perm, lock and
  level files, bones, `save/`, paniclog, `dumplog/`): `nethack` under
  `$XDG_DATA_HOME`, else `~/.local/share/nethack`, made with its empty
  files on the first run (`av_playground()`). `include/unixconf.h` makes it
  NetHack's own `VAR_PLAYGROUND` when `AV_PATHS` is defined, so upstream's
  prefix machinery (`fqname()`) does the rest.
- `sys/unix/unixmain.c`: `chdirx()` uses `av_hackdir()` where it used
  `HACKDIR`. `NETHACKDIR`, `HACKDIR` and `-d` still pick another
  playground, which then holds everything, as upstream.
- `src/cfgfiles.c`: without `~/.nethackrc` (still read first, as every
  NetHack player expects), the options file is `nethackrc` in the player's
  folder, which is also where `#saveoptions` then writes.
- `src/windows.c`: a relative `DUMPLOGFILE` is in the player's folder, and
  `sys/unix/sysconf` turns the end-of-game dump on as
  `dumplog/<name>.<date>.txt` (NetHack's morgue file).

Upstream changes are guarded by `#ifdef AV_PATHS`, defined only by
`asciiverse/hints`.

## Build

- `asciiverse/hints`: a hints file of our own (from `linux-minimal` and
  `linux.500`) for one binary with both the tty and curses interfaces,
  tty the default; `OPTIONS=windowtype:curses` picks curses. At 100x30
  tty is the better default: curses squeezes the message window into a
  24-column box beside the map. Also `DLB` (one `nhdat` file),
  `SELF_RECOVER`, `DUMPLOG`, saves compressed with zlib linked in
  (`ZLIB_COMP`) instead of running `/usr/bin/compress`, and crash reports
  off (`NOCRASHREPORT`, `NOPANICTRACE`: upstream would open a browser at
  nethack.org with `xdg-open` or `open`, or run gdb). On Linux
  `_GNU_SOURCE`, which Alpine's fortify headers need before `curses.h`.
- `asciiverse/build.sh`: fully static Linux in Alpine (musl, ncursesw and
  zlib linked in), macOS per architecture (`CC_ARCH`, system ncurses and
  zlib). Lua 5.4.8, which NetHack 5 runs its levels in, is fetched from
  lua.org, checked against upstream's `submodules/CHKSUMS` (the build stops
  on a mismatch) and linked in statically as `lib/lua/liblua-5.4.8.a`, as
  upstream's `make fetch-lua` does. Leaves the install in `out/`. The
  Guidebook is not rebuilt (it needs nroff); the archive ships upstream's
  `doc/Guidebook.txt`.
- `asciiverse/nfpm.yaml`: .deb, .rpm, .apk and Arch packages:
  `/usr/bin/nethack`, data in `/usr/share/nethack`, Guidebook and licence
  in `/usr/share/doc/nethack`, the man page. Nothing setgid, no menu entry.
- `asciiverse/record-preview.py`: records `asciiverse/nethack.cast`, a
  Valkyrie exploring by breadth-first search over the screen (pyte), best
  of several takes.
- `.github/workflows/release.yml`: builds all of it on a tag `v5.0.0-av.<n>`.
- `.gitignore`: `out/` and `dist/`.

## Not built

- `recover`: not needed with `SELF_RECOVER`; the game restores an
  interrupted game itself.
- The X11, Qt and Windows GUI interfaces, sound libraries, `sfctool`.
- Windows: upstream supports a Windows console build (`sys/windows`,
  `consoletty.c`; `GNUmakefile` under MSYS2/MinGW, see
  `sys/windows/build-msys2.txt`, or Visual Studio). It keeps its writable
  files in per-user folders already (see `PORTABLE_DEVICE_PATHS` in
  `sys/unix/sysconf`). A MinGW job in the release workflow building
  `nethack.exe` with the tty port is what it would take; not tried yet.

## Known upstream quirk

Keys typed ahead very fast between the "Is this ok?" menu and the opening
text can log `nhassert(ttyDisplay->toplin == TOPLINE_EMPTY)` to the
paniclog (`win/tty/wintty.c`); seen only with scripted input, not at
human speed. Not ours to change.
