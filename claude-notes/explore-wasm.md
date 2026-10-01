# Exploration: a WebAssembly port (2026-10-01)

An exploration, **not a decision**. Jack's question: if we wanted to play SMIL
documents inside a browser window via WebAssembly, how much of the core would
survive, and is the current architecture still feasible?

## Verdict

Almost all of the core survives, and the architecture suits it well: the
changes fall exactly where the design made things replaceable.

## Survives unchanged

- **The SMIL engine** (`smil2`: timing, scheduler, layout, animation,
  transitions, smilText, State) and the DOM/document code in `lib`. Portable
  C++; for Emscripten this is mostly a build question.
- **The XML parser:** expat compiles to WebAssembly. Alternatively, the
  browser's `DOMParser` could feed the tree builder.
- **Plugins:** `plugin_engine.cpp` already has a `WITH_STATIC_PLUGINS`
  path; in the browser, plugins are linked statically.

## Replaced, behind existing interfaces

1. **Threading via the event processor.**
   - Threads are created in one place (`lib/unix/unix_thread.cpp`).
   - `lib/event_processor.cpp` already has two implementations: a thread
     with a priority queue, and a GCD/libdispatch variant (Bo Gao, 2011).
   - A third variant **driven by the browser event loop**
     (`setTimeout`/`requestAnimationFrame` via Emscripten) fits next to
     them: single-threaded, locks become no-ops.
   - The alternative is Emscripten pthreads (Web Workers), which needs
     COOP/COEP headers on every embedding page (a hosting restriction).
2. **Datasources:** a `fetch`-based datasource replaces `posix`/`stdio`,
   behind `datasource_factory`.
3. **Renderers: a new "web" back-end**, a peer of `cg`/`gtk`/`d2`/`SDL`.
   - Regions become positioned elements, and media become
     `<img>`/`<audio>`/`<video>` elements: let the browser decode and play.
   - Text becomes HTML/CSS, and transitions become CSS or canvas.
   - This matches the 2003 design text ("integrating third-party tools":
     pass a URL and say play).
   - No ffmpeg in WebAssembly needed.
   - The **`timer_sync`** mechanism (2012) synchronises the SMIL clock with
     `<video>.currentTime`.

## Browser-specific traps

- **Autoplay policy:** audio only after a user gesture, so "click to start"
  or start muted.
- **CORS:** media elements may load cross-origin media without CORS;
  fetching raw bytes may not. Another argument for DOM elements over
  decoding ourselves.
- **Clock precision:** media `currentTime` is coarse and irregular, so
  clock synchronisation needs smoothing.

## Relation to other ideas

Like the "Python skin + offscreen C++ renderer" idea (see the discussion
around D3), this makes the **renderer back-end the boundary** between the
timing core and whatever presents it. The per-toolkit back-ends
(`cg`/`gtk`/`d2`/`SDL`) were the 2010 answer; a web back-end (and possibly
one offscreen back-end for the desktop) would be the 2026 answer.

Possible prior art (HTML+TIME in Internet Explorer, INRIA's JavaScript
timesheets) not investigated; Jack knows that landscape better.

## Testing resource: the official SMIL 3.0 test suite

The W3C SMIL 3.0 test suite is still online:
<https://www.w3.org/2007/SMIL30/testsuite/> (zip:
`New-SMIL30/New-SMIL30-testsuite-16-09-2008.zip`; per module: Timing and
Sync, Animation, Layout, Media, Content Control, smilText, State,
Structure, Metainformation, Namespace/Doctype; plus SMIL 2.0 tests). The
implementation report
(<https://www.w3.org/2007/SMIL30/SMIL30-implementation-result.html>) records
Ambulant's 2008 results per test.

Uses:

- conformance tests for any revived core (compare against the 2.6
  reference player and the 2008 results);
- a concrete yardstick for the WebAssembly-port vs TypeScript-rewrite
  question: "how much of SMIL is needed" can be expressed per test-suite
  module, and a rewrite has to pass the same tests.

Not downloaded yet.
