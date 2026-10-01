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

## D2 — Don't revive bgen; Python bridge out of scope (2026-10-01)

**Decision:**

- Don't revive bgen (Python 2, removed from Python 3) or the generated
  `src/pyambulant` bridge.
- The Python bridge and the Python plugins are **out of scope** for now.
- The one functional dependency, the trace plugin that produced the
  nightly reference traces (`tests/nightly`), is replaced by a small C++
  `player_feedback` implementation in the headless driver.
- If Python access is wanted later: pybind11 or nanobind, with trampoline
  classes for a **curated** set of interfaces (e.g. `embedder`,
  `player_feedback`, `playable_factory`, `state_component`), not the whole
  API.

**Why:** Jack's criterion: reviving bgen is only worth it if something
important depends on two-way bridging, i.e. implementing first-class
citizens in Python. Otherwise pybind11 or even ctypes is easier and more
modern. Claude checked what depends on it:

- Every Python plugin subclasses a C++ interface, so they all need the
  reverse bridge. But apart from `pyamplugin_trace` (`embedder`,
  `player_feedback`) they're examples or experiments: `DummyPlayable…`,
  `DummyRecorder…`, `MyTimerSync…`, `MyStateComponent…`. The real SMIL
  State engine is the C++ `xpath_stateplugin`.
- pybind11/nanobind can also bridge both ways (trampolines for virtual
  methods), so two-way bridging no longer requires bgen. What bgen offered
  extra was automatic generation from the headers, which isn't needed for a
  small curated set.
- ctypes is unsuitable: C only, so it would need a C API shim, and
  implementing interfaces in Python would mean hand-written callbacks.

Background, including the bgen history: [players-and-python.md](players-and-python.md).
