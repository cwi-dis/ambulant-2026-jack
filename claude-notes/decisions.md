# Decisions

Revival decisions with their reasoning, newest last.

## D1 — Minimal scope, modern foundations (2026-10-01)

**Decision:** keep the scope minimal, but don't keep the historical
tooling:

- **Dependencies:** current versions from system packages (Homebrew on
  macOS, apt on Linux, found through pkg-config). No pinned historical
  versions, no `build-third-party-packages.py`.
- **Build system:** CMake, plus GitHub Actions CI from the start. The
  existing autotools, `projects/xcode43` and `projects/vc10` files stay
  **untouched** (neither maintained nor deleted) until CMake covers the
  chosen scope. Then they're removed.
- **Initial scope:** `libambulant` core, a headless driver (timing-trace
  comparison against `tests/nightly`), and the SDL2 player, on macOS and
  Linux. Native players (Cocoa/cg, GTK, Direct2D), iOS, Android, Python
  bindings and browser plugins are out of scope for now.

**Why:** Jack framed it as two extremes. "Minimal" would keep the historical
tools and packages; "complete" would mean CMake, CI, decent dependency
management. Claude's analysis, agreed by Jack:

- Historical dependencies aren't the cheap option. Python 2 is gone from
  package managers; ffmpeg 2.0 would have to be built from source on arm64
  with a current clang (assembly and configure patches); the Xcode 4.3 and
  VS2010 projects assume old architectures and SDKs. Porting to current
  ffmpeg (~30 call sites in 5 files) is less work than keeping 2.0 alive,
  and the result is something we'd keep.
- Any effort spent reviving the 1,100-line `configure.ac` and the old IDE
  projects would be thrown away once we move to CMake.
- "Minimal" makes sense for **scope**, and that's what keeps the effort
  down.

**Preceded by a probe:** compile the core with the *existing* autotools and a
current compiler, time-boxed to about an hour, in a scratch copy, with no
fixes committed. This separates code rot (compiler errors) from build-system
rot, before the switch to CMake makes the two indistinguishable. Results:
[probe-autotools.md](probe-autotools.md).
