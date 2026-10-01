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

## D3 — What not to port (2026-10-01)

**Decision:** classify the tree into four groups. Groups A and B are removed
**together, once CMake covers the D1 scope**. Not earlier, because removing
directories breaks the autotools build, and D1 leaves autotools untouched
until then. Groups C and D stay.

**A. Pointless: dead technology.**

- `src/npambulant`, plus the npambulant projects in `projects/xcode43` and
  `projects/vc10`, xulrunner in the third-party script, and
  `scripts/geniepluginwebpage.py` (NPAPI is gone from all browsers)
- `src/ieambulant` plus its `vc10` projects (ActiveX / Internet Explorer)
- `gui/gstreamer`, src and include (gstreamer 0.10, audio only)
- `gui/SDL/sdl_pango_smiltext.cpp` (SDL_Pango is dead; the SDL2_ttf variant
  remains)
- `gui/cg/atsui_text.mm` (ATSUI; unreferenced dead code)
- `plugins/wkdombridge.mm` and `wkdomplugin.cpp` (legacy WebKit `WebView`
  Objective-C DOM API)
- `sandbox/gst-rtsp-0.10.8` and `sandbox/gstambulant-plugin-package-0.10.0`
  (168k lines of vendored gstreamer 0.10 code)
- `sandbox/sdl2-test`, `tracevis`, `sdl-fullscreen`, `delaying_webserver`
- `installers/mkubuntuppa.sh`, `ambulant.spec.in`
- `Documentation/mediasupport` (the 2015 codec support matrix, mostly
  generated; Python 2 generators)

**B. Superseded by D1:**

- autotools files, `projects/xcode43`, `projects/vc10`
- `third_party_packages/`, `scripts/build-third-party-packages.py`
- `scripts/nightlybuild` (replaced by GitHub Actions)
- `ffmpeg-osx-fatbuild.sh`, `internalize_*`, `patch_libtool.sh`,
  `precheck-*`
- `debian/` and `installers/sh-macos` are to be redone later, not ported

**C. Not now, but not pointless (stay in the tree):**

- `player_iphone` and the UIKit parts of `gui/cg`
- `projects/android`
- `player_mfc` + `gui/d2`
- `player_gtk` + `gui/gtk`
- `pyambulant` and the Python plugins (D2)
- the streaming line (`ambulant-recorder-plugin`, `ambulant-server`,
  `gstambulantsrc-1.0.x`)

**D. Keep:** the core, `gui/cg`, `gui/SDL`, `player_sdl`, `player_macosx`,
the C++ plugins, `Extras` (including `DTDCache`, used by Xerces), `tests`,
`Documentation/Design` and `Documentation/user`.

**Why:** fewer moving parts for the CMake work, less noise when searching
the code. Nothing is lost: everything remains in git history (and in the
hg repositories).

## D4 — Direction: an embeddable SMIL engine for accessibility (2026-10-01)

**Status: a direction, not a final decision.** Jack: "we don't know yet"
which hosts exactly.

**Context (Jack):** the goal is to use SMIL for multimodal presentation for
people with disabilities (visual or hearing impairments). The web is one
platform of interest; Unity VR applications are another; maybe Unreal or
others.

**Direction:**

1. **Port, don't rewrite.** A TypeScript rewrite would only serve the web;
   Unity would need C# and Unreal C++, so three engines. A portable C++
   core serves all of them: web via WebAssembly; Unity via a native plugin
   with a C API (P/Invoke; Unity WebGL builds compile native plugins to
   WebAssembly too); Unreal by linking C++ directly.
2. **The host drives the clock.** Unity `Update()`, Unreal `Tick`, browser
   `requestAnimationFrame`: one externally driven, single-threaded event
   processor variant (next to the existing thread and GCD variants).
   `timer_sync` aligns the SMIL clock with host media clocks.
3. **Presentation is an abstract interface.** The core doesn't draw. It
   tells the host to start, stop, position or animate media in regions,
   and the host presents them with its own means (Unity
   `VideoPlayer`/`AudioSource`, Unreal Media Framework, HTML elements). In
   VR a region can be a 3D panel, a spatial audio source, or a caption
   surface. This generalises the 2003 design's "pass a URL and say play";
   the `none`/trace back-end is the closest existing starting point.
4. **A narrow C API** as the single external interface: open a document,
   set preferences (captions, audio description, language), advance time,
   receive presentation events, send user events. Any language binding,
   Python via `ctypes` included, becomes small.

**Why it fits:** SMIL has accessibility built in, and Ambulant implements
it: `smil2/test_attrs.cpp` handles `systemCaptions`, `systemAudioDesc`,
`systemOverdubOrSubtitle`, `systemLanguage` and `customTest`, alongside
smilText (captions) and SMIL State (user preferences). DAISY people were
among the SMIL 3.0 editors.

**Effect on earlier decisions:**

- **D1** stays (CMake, current dependencies), but the minimal product
  becomes "Welcome plays through the embeddable engine with a simple
  host". That host could be the SDL window, the browser or a small Unity
  scene.
- **D2** stays (no bgen); a C API makes a later Python binding small.
- **D3** stays.
- The native back-ends (`cg`, `gtk`, `d2`, `SDL`) become optional desktop
  hosts instead of the centre of the design. Jack's "Python skin" idea is
  one more host.
