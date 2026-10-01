# Probe: existing autotools build with a current toolchain (2026-10-01)

Purpose (see [decisions.md](decisions.md) D1): find out how much of the rot
is in the **code** and how much in the **build system**, before switching
to CMake. Done on an exported copy of HEAD in Claude's scratchpad. Workarounds
were applied only there, and **nothing was committed**.

## Environment

macOS 26 (SDK 26.5), arm64, Apple clang 21. Homebrew: autoconf 2.73,
automake 1.19, libtool 2.6.2 (as `glibtoolize`), gettext 1.0, pkg-config
3.0.7, expat 2.7.4, ffmpeg 9.0 (libavcodec 63), SDL2 via `sdl2-compat` 2.32
(SDL2 API on top of SDL3). Not installed: SDL2_image, SDL2_ttf.

autoconf, automake and libtool weren't installed at first; Jack installed
them. `autogen.sh` already knew about `glibtoolize`, so Homebrew's
g-prefix wasn't a problem.

Wall clock for the probe itself: about 4 minutes (15:03–15:07), plus waiting
for the tool install.

## Results

### Build-system rot (autotools)

| # | Problem | Probe workaround |
|---|---------|------------------|
| B1 | `autogen.sh` hard-codes `$prefix/share/aclocal/gettext.m4`; gettext ≥ 0.24 installs its macros in `share/gettext/m4` | add that directory to `ACLOCAL_FLAGS` |
| B2 | Bundled `libltdl` subdirectory wants to regenerate its `aclocal.m4` with `aclocal-1.18`, which doesn't exist (automake is 1.19) | none (see B3) |
| B3 | Using Homebrew's system `libltdl` instead fails: the 2009-era `LTDL_INIT` checks require a `libltdl.la`, which Homebrew doesn't ship ("invalid ltdl library directory") | `--without-ltdl-plugins` (no plugin loading) |
| B4 | `po/`: "gettext infrastructure mismatch: Makefile.in.in from gettext 0.19 but macros from 0.24" | ignored (only i18n) |
| B5 | `configure.ac`: obsolete-macro warnings (`AC_ISC_POSIX`, `AC_HEADER_STDC`, `AC_PROG_LIBTOOL`, `AC_TRY_LINK`) | harmless for now |
| B6 | SDL configuration: `--with-sdl2-frontend` without SDL2 *video* (which needs an extra `-DWITH_SDL_VIDEO`, per `sandbox/ambulant-server/README`) produces a player that doesn't compile (2 missing factory functions) | not pursued |

So autotools *can* still be made to work, but only with workarounds at
every step, and plugin loading (central to Ambulant's design) doesn't work
without further surgery on libltdl.

**Clarification (after a question from Jack): libltdl is only partly a
build-system problem.** B2 and B3 are autotools machinery and disappear with
CMake (link against the system libltdl, or drop it). But the *code* is tied
to libtool too: `common/plugin_engine.cpp` discovers plugins by scanning for
**`*.la`** files (line 269), libtool's archive files, and opens them with
`lt_dlopenadvise()`. A CMake build produces no `.la` files, so plugins would
silently not be found. Switching to CMake therefore needs a small code change:
scan for `.dylib`/`.so`. At that point it's simplest to replace `lt_dl*` with
POSIX `dlopen`/`dlsym` (`lt_dladvise_global` = `RTLD_GLOBAL`). Windows
already uses `LoadLibrary` in the same file. About 30 lines; a separate
commit, since it changes plugin discovery.

### Code rot: core (`libambulant` without ffmpeg/SDL)

**Everything compiles except one file**, and `libambulant.dylib` links
(with `--without-ltdl-plugins`).

- `net/stdio_datasource.cpp`: 3 errors, `if (file >= 0)` on a `FILE *`.
  An ordered comparison of a pointer with 0 was always a bug; modern clang
  rejects it. Fix: `!= NULL`. (Applied in the scratch copy only.)
- Warnings: 44 `sprintf`/`vsprintf` deprecations, and **one real latent
  bug**: `net/url.cpp:814` (`result = get_file().c_str();` keeps a pointer
  into a destroyed temporary). Line 809 has the same pattern
  (`result = abs_path.c_str();` with `abs_path` going out of scope). This
  is a use-after-free in local-file URL resolution that has presumably
  "worked" by luck.

### Code rot: ffmpeg (current ffmpeg 9.0)

| File | Errors |
|------|-------:|
| `net/ffmpeg_common.cpp` | 29 |
| `net/ffmpeg_video.cpp` | 23 |
| `net/ffmpeg_audio.cpp` | 16 |
| `net/ffmpeg_raw.cpp` | 0 |

All are known ffmpeg API removals: `AVStream::codec` (→ `codecpar` plus
your own `AVCodecContext`), `avcodec_decode_video2`/`audio4` (→
send/receive API), `av_free_packet`, `avcodec_alloc_frame`,
`av_register_all`, `CODEC_ID_*` (→ `AV_CODEC_ID_*`), `PIX_FMT_*`/`PixelFormat`
(→ `AV_PIX_FMT_*`/`AVPixelFormat`), `AVPicture`, `channel_layout` (→
`AVChannelLayout ch_layout`), `pkt_pts`, `best_effort_timestamp`. That
confirms the rot-audit estimate: a well-understood, contained port in 3
files. The decode-loop change (send/receive) is the only part that touches
control flow.

### Code rot: SDL (sdl2-compat)

- `include/ambulant/gui/SDL/sdl_window.h:50`: one line, C++11 narrowing
  (`unsigned` → `int` in `SDL_Rect` brace init). Trivial.
- Nothing else in the SDL back-end failed to compile in this
  configuration. (SDL2 video wasn't enabled, see B6, so the video renderer
  wasn't compiled.)

## Conclusions

1. **The code is in much better shape than the build system.** The core
   needs a 3-line fix to compile with a 2026 compiler. ffmpeg needs a real
   but contained port (≈ 70 errors in 3 files). SDL needs one line.
2. **The build system is where the workarounds pile up** (B1–B6), and
   plugin loading doesn't work. This supports decision D1: go to CMake
   instead of repairing autotools.
3. **The url.cpp dangling pointers are a real bug** that should be fixed
   regardless (a separate commit, as a behavioural fix).
4. Not tested: actually *running* anything. There's no headless driver
   yet (see [rot-audit.md](rot-audit.md)).
