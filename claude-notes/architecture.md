# Ambulant architecture (cold pass)

## What it is

Ambulant Player is a playback engine for SMIL 3.0 (and 2.1/2.0) multimedia
presentations, written in C++ with a full Python binding. Last version in the
tree is 2.7 (nightly); 2.6 was the last stable release (early 2015). License
LGPL 2.1. *(docs: README, NEWS)*

Stated design goal: a **research system** in which every component can be
replaced (scheduler, network, renderers) without touching the rest. This is
achieved through abstract interfaces + factories everywhere. *(docs:
Design/text/overall.txt)*

## Size

About 1,650 tracked files. Rough line counts for C/C++/ObjC/Python *(code)*:

| Area | Lines | Notes |
|------|------:|-------|
| `src/libambulant/smil2` | 13.4k | SMIL semantics: timing, scheduling, layout, animation, transitions, smilText |
| `src/libambulant/gui/*` | 21.6k | Renderer back-ends: SDL 5.9k, d2 (Direct2D) 5.7k, cg (CoreGraphics) 4.5k, gtk 4.4k, gstreamer 0.6k, none 0.3k |
| `src/libambulant/lib` | 6.6k | Runtime: DOM, XML parsers, event processor, timers, threads, logging |
| `src/libambulant/net` | 6.3k | Datasources: file, posix, win32, ffmpeg demux/audio/video |
| `src/libambulant/common` | 3.6k | Glue: factories, gui_player, plugin engine, regions, preferences, state |
| `include/ambulant` | ~8k+ | Headers for all of the above (all public headers live here) |
| `src/pyambulant` | 37k | Python bridge, mostly **generated** by `bgen` (in-tree) |
| `src/plugins` | 3.9k | Example and optional plugins |
| Players | 2–3.7k each | gtk, sdl, macosx, iphone, mfc |
| Browser plugins | 3.3k + 0.9k | npambulant (NPAPI), ieambulant (ActiveX) |

So the platform-independent engine is roughly 30k lines; the rest is
per-platform rendering, players, and bindings.

## Layers and namespaces *(docs + code)*

Everything is in namespace `ambulant`, with sub-namespaces that match
directories:

- `ambulant::lib` — basic runtime: `node`/`document` (DOM), `event_processor`,
  `timer`, `thread`, `critical_section`, `logger`, `refcount`, XML parser
  adapters (expat built-in, Xerces and libxml2 as plugins).
- `ambulant::net` — `url`, `datasource` hierarchy (raw, audio, video, packet),
  `datasource_factory`, ffmpeg-based demuxing and decoding.
- `ambulant::common` — interfaces tying it together: `player`, `playable`,
  `renderer`, `factories`, `gui_player`, `layout`/`region`, `embedder`,
  `plugin_engine`, `state_component` (SMIL State).
- `ambulant::smil2` — the SMIL implementation: `smil_player`, `timegraph`,
  `time_node`, `scheduler`, `smil_layout_manager`, animation (`animate_*`),
  transitions, `smiltext`.
- `ambulant::gui::<toolkit>` — renderers and window/surface implementations
  for one toolkit each.

Platform-specific code goes in `unix`/`win32` sub-namespaces; platform ifdefs
use `AMBULANT_PLATFORM_*` macros. `include/ambulant/config` is a **boost-style
config system** (compiler/platform/stdlib selection headers) — that's where all
the "boost" grep hits come from; there's no real Boost dependency. *(code)*

Code conventions (tabs, `m_` members, `snake_case`, `_impl` for default
implementations, `detail` namespace for semi-private classes) are in
`Documentation/Design/text/organisation.txt`.

## Core objects and playback flow *(docs: walkthrough.txt, last updated for 1.8; spot-checked against headers)*

1. **Embedder / main program** (one per player app) creates a `gui_player`
   subclass per document (often called `mainloop`).
2. `gui_player` (via the `factories` base) populates the factories:
   `window_factory` (from the app), `global_playable_factory` (renderers),
   `datasource_factory`, `parser_factory`, `node_factory`, plus later
   additions `state_component_factory`, `timer_sync_factory`,
   `recorder_factory` *(code: common/factory.h)*.
3. `init_plugins()` — `plugin_engine` loads dynamic plugins, which register
   extra factories (this is how ffmpeg, Xerces, Python, state plugins get in).
4. The document is parsed into a `lib::document` (DOM of `lib::node`).
5. `create_smil2_player(document, factories, embedder)` creates the
   `smil_player`, which builds:
   - a `timer` (master clock) and `event_processor` (priority run-queue in its
     own thread — **the engine's heartbeat**; everything that waits does so by
     posting an event),
   - a `smil_layout_manager` (region tree → `surface_template` tree, via
     `window_factory`),
   - a `timegraph` (the `<body>` as a tree of `time_node`s) and a scheduler.
6. On `start()`, the scheduler walks the timegraph. When a media node must
   play, `global_playable_factory::new_playable()` asks each registered
   factory in turn until one accepts. The renderer gets a surface from the
   layout manager, opens a `datasource`, receives data via event callbacks,
   asks for redraw (`need_redraw` up the surface tree, `redraw` back down),
   and reports `started`/`stopped` back to the scheduler through
   `playable_notification`.

Memory management: hand-rolled intrusive refcounting (`lib::ref_counted`),
used only where objects are shared across components. *(docs)*

## Renderer back-ends (`src/libambulant/gui`) *(code)*

| Back-end | Platforms | Status in 2015 *(guess, from activity)* |
|----------|-----------|------------------------------------------|
| `cg` | macOS (Cocoa + CoreGraphics) and iOS (UIKit) | active |
| `SDL` | Linux/Android/macOS; SDL2 | active; newest |
| `gtk` | Linux, GTK 2 or 3 | active |
| `d2` | Windows, Direct2D + DirectShow | active |
| `gstreamer` | Linux, gstreamer 0.10 | probably stale |
| `none` | anywhere, renders nothing | useful for headless tests |

Earlier back-ends deleted along the way: `cocoa` (→ `cg`), `dx` (→ `d2`),
`qt`, `dg`. *(code: deleted files in history)*

Note the `.m`/`.mm` file pairs in `gui/cg` and `player_macosx`: the `.m` is a
one-line `#include` of the `.mm`, a workaround for automake not handling
Objective-C++. Not a conversion artifact. *(code)*

## Players and embeddings

- `player_gtk` → `AmbulantPlayer_gtk` (Linux desktop)
- `player_sdl` → `AmbulantPlayer_sdl` (Linux, Android via `projects/android`)
- `player_macosx` (Cocoa app, Xcode project in `projects/xcode43`)
- `player_iphone` (iOS, "preliminary")
- `player_mfc` (Windows MFC app, VS2010 projects in `projects/vc10`)
- `npambulant` — NPAPI browser plugin (Firefox/Safari/Chrome, all platforms)
- `ieambulant` — ActiveX control for Internet Explorer
- `pyambulant` — Python module; with `player_pygtk` example

## Plugins (`src/plugins`) *(code)*

`plugin_ffmpeg` (all ffmpeg media), `xercesplugin` (validating parser),
`pythonplugin` (load plugins written in Python), `xpath_stateplugin` (SMIL
State with XPath via libxml2), `wkdomplugin` (WebKit DOM bridge, macOS),
`plugin_timesync` (distributed synchronisation), `rot13plugin` and
`dummy_stateplugin` (examples), and Python plugins `pyamplugin_*`.

## Build systems

- **autotools** (`configure.ac`, `Makefile.am`) for Linux and also macOS
  command-line builds. Many `--with-*` options pick back-ends and deps.
- **Xcode 4.3-era projects** in `projects/xcode43` (macOS, iOS, npambulant).
- **Visual Studio 2010** in `projects/vc10`.
- **Android** project in `projects/android/AmbulantSDLPlayer`.
- Third-party deps are fetched and built by
  `scripts/build-third-party-packages.py` (pinned versions, hard-coded URLs).
- Packaging: `debian/`, `ambulant.spec.in` (RPM), `installers/`
  (NSIS, macOS shell, Ubuntu PPA, iOS).

## Testing

- `tests/nightly`: reference traces (`*-reference.txt`) of `TEST` lines
  (`node_started`/`node_stopped` with timestamps), produced by the
  `pyamplugin_trace` Python plugin while playing a document. Only two
  reference files exist (Welcome, VideoTests-http). *(code)*
- `scripts/testoutputcompare.py`, `scripts/autotest.linux.sh`,
  `scripts/nightlybuild` — the nightly-build infrastructure.
- `Documentation/misc/Testing.txt`, `TestMatrix.ods`, `Run-tests.sh` — manual
  test procedures.
- A larger body of test documents lived in a separate `ambulant-documents`
  repository *(docs: README-hg)* — not in this repo.

## Design documentation worth knowing about

`Documentation/Design/index.html` (reStructuredText sources in `text/`,
OmniGraffle diagrams with PDF exports in `models/`). Most texts say "last
updated for 1.8" (~2007), so treat details with care; the overall structure
still matches the headers.
