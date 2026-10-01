# Recovering lost history and issues (deferred)

Decision (Jack, 2026-10-01): **don't** bring hg tags, branches or the
SourceForge issues into this fork now. They aren't needed for the porting
investigation. This file records what exists and how to import it later, so
the decision can be revisited cheaply. Background: the "old repositories"
section of [history-jack.md](history-jack.md).

## Sources

| What | Where | Notes |
|------|-------|-------|
| Full hg history (9,162 changesets, 160 branches, 43 tags) | Jack's `~/src/ambulant` | **Most complete copy known.** The SourceForge mirror stops around May 2014 |
| Test/demo documents (hg, 149 changesets) | Jack's `~/src/ambulant-documents` + `hg.code.sf.net/p/ambulant/ambulant-documents` | SF has 1 extra changeset (2011-09-19) |
| Sandbox (CVS, 2005–2009) | Jack's `~/src/ambulant-sandbox` | SourceForge CVS service is gone, so possibly the only copy |
| Private repo (hg) | Jack's `~/src/ambulant-private` | contains secrets; **never publish** |
| Issue tracker (887 tickets) | SourceForge `p/ambulant/bugs` | also `support`, `reviews`, `mailman` tools there |

**Backups (Jack):** not a real risk. Jack's `~/src` on his work desktop is
Time Machine backed up, and older copies exist on older machines.

## How to import the hg history later

Goal: add the 1,987 missing changesets and the tags **without changing
the hashes of the 7,175 commits already on GitHub**, because every fork
(including Karthik's) is based on them.

1. **Map hg nodes to existing git commits.** git's history is exactly
   `::default` in hg, so match each hg changeset in `::default` with a git
   commit on (author date incl. timezone, author, full message), and check
   that the mapping is 1:1 for all 7,175. (Not done yet; the counts match
   exactly, but there may be ties on identical timestamp+message, which need
   to be resolved by parent order.)
2. **Convert only the missing part.** Use `hg-fast-export` (frej/fast-export)
   or a small custom exporter over `not ::default`, emitting git
   fast-import commands where parents in `::default` are referenced by their
   existing git SHA (`from <sha>` / `merge <sha>`) instead of being
   re-exported. Each hg named branch becomes a git ref, e.g.
   `refs/heads/hg/<branchname>`, or better `refs/hg/<branchname>` so that
   the 160 refs don't clutter the GitHub branch list.
3. **Tags:** create git tags from `.hgtags`, pointing at the mapped (or
   newly imported) commits. Annotated tags with the original hg date and
   tagger. Skip the meaningless ones (`release-ambulant-18-merge` and
   `-merge-trunk` were declared meaningless by Kees in 2006).
4. **Verify:** for a few tagged releases, compare the tree of the git tag
   with `hg archive -r <tag>`.
5. **Optionally** add a `.hgtags`-derived note or `git notes` with the
   original hg revision numbers (commit messages refer to them, e.g.
   "merge 9087").

Branches most likely to matter later, if ever: `release-ambulant-NN-branch`
(the code as released), `amis-release-30` (Marisa's AMIS DAISY reader
release), `exp-kees-plugin-ncl`, `exp-kees-sdl2-mac`.

## How to archive the issues later

Plan: a **read-only archive** (no import into GitHub Issues).

1. If Jack is still a SourceForge project admin, the simplest is SF's
   **project export** (Admin → Export), which gives a JSON backup of the
   tracker including discussion threads and attachments.
2. Otherwise use the public REST API:
   - list: `https://sourceforge.net/rest/p/ambulant/bugs/?limit=500&page=N`
     (`count` = 887),
   - per ticket: `https://sourceforge.net/rest/p/ambulant/bugs/<n>/`
     (fields, status, labels, `discussion_thread` with posts, attachment
     URLs).
3. Store the raw JSON plus a rendered Markdown file per ticket (and an index
   by status and date) in a separate repo or directory, e.g.
   `ambulant-issues-archive`. Map old 7-digit SF IDs (used in commit
   messages before the renumbering) to new numbers, if the export contains
   them.
4. Do the same for the `support` tracker and the mailman list archives if
   they turn out to have content.

## Binary releases (found 2026-10-01)

SourceForge `projects/ambulant/files/` still has 58 files: binaries for every
release from 1.2 (Zaurus, 2004) to 2.6 (2015). For 2.6: macOS `.dmg`,
Windows `.exe`, source tarball, npambulant for Linux/macOS/Windows, and
ieambulant. Older: 2.4, 2.2, 2.0.2 (WinCE, Nokia 800), 1.8 (Windows, Nokia
770), 1.4.5 (Mac OS X 10.3), plus prebuilt ffmpeg/third-party bundles for
Windows.

- **`Ambulant-2.6-mac.dmg`** (inspected, not run): universal x86_64 + i386,
  minimum macOS 10.7, SDK 10.10, signed with CWI's Developer ID
  (W5J7983J99, 2015-02-02), not notarized. Bundles SDL2, ffmpeg 2.0 (lavc
  55), expat, libambulant, libambulant_cg/_sdl/_ffmpeg; `PlugIns/` has
  python, xerces and xpath_state plugins **with their `.la` files** (as
  `plugin_engine.cpp` expects). The Python plugin links against the system
  Python 2.7 framework, which no longer exists.
- **"2.6 for MacOSX10.6"** (uploaded 2017-08-23): a hand-built 2.6 for
  10.6.8, because the official build didn't work there. Its README documents
  the workarounds (yasm 1.2, ffmpeg 2.0.2 with `--disable-optimizations`
  because of a compiler bug, an SDL2 patch from MacPorts, signing on a
  machine that still had the keys).
- **Running old binaries:** this machine (boor) has Rosetta 2. Jack also has
  beignet (a 2018 Intel MacBook) and flap (older than Ambulant 2.6) for
  running the x86_64 builds natively.

### Reference player: 2.6 for macOS runs (2026-10-01)

**Jack:** mounted `Ambulant-2.6-mac.dmg` on boor (macOS 26, Apple Silicon,
via Rosetta 2) and double-clicked Ambulant Player: **it opens without a
hitch and plays Welcome.** Audio plays and stays in sync with the images;
the clickable link works (it opens a browser on ambulantplayer.org, which
returns an nginx error page, as expected); play/stop/pause and the menus
work. All three demo documents (in `DemoPresentation`) seem to work.

Log from the player's logging window (abridged):

```
DEBUG Ambulant Player: compile time version 2.6, runtime version 2.6
DEBUG Ambulant Player: built on Feb  2 2015 for Macintosh/CoreGraphics/x86_64
TRACE plugin_engine: using LTDL plugin loader
TRACE plugin_engine: Scanning plugin directory: .../Ambulant Player.app/Contents/PlugIns
TRACE plugin_engine: examining Python plugin libamplugin_xpath_state.la
TRACE plugin_engine: loading .../libamplugin_xpath_state.la
TRACE plugin_engine: examining Python plugin libamplugin_xerces.la
TRACE plugin_engine: loading .../libamplugin_xerces.la
TRACE plugin_engine: examining Python plugin libamplugin_python.la
TRACE plugin_engine: Done with plugin directory: ...
TRACE xerces_plugin: registered
TRACE Using parser any
TRACE xpath_state_plugin: registered
TRACE file:///.../Welcome.smil: Parsing document...
TRACE file:///.../Welcome.smil: Parser done
TRACE surface_impl[...].renderer_done(...): not found in 0 active renderers!
```

**Claude's reading:**

- It's the **native Cocoa player** (`player_macosx` + `gui/cg`), not a GUI
  toolkit on top of SDL, as Jack remembered it. SDL2 and `libambulant_sdl`
  are bundled only for audio output.
- Plugin loading works exactly as the code says (scan for `.la`, load via
  libltdl). The log message labels every plugin "Python plugin", a small
  bug in the message. The Python plugin is examined and silently skipped,
  presumably disabled by default.
- `renderer_done ... not found in 0 active renderers!`: a harmless warning.

**Consequences:** we have a **reference player** for behaviour comparison.
And the native Cocoa player is a candidate for the minimal product, instead
of the bare SDL window, since it demonstrably still works as a binary. To
be decided when the MVP plan is picked up again.

## Copies on sap (Jack's home machine, found 2026-10-01)

The sources above are on boor (Jack's work desktop). Sap has an overlapping
but different set, inspected read-only:

| Directory | Size | What | Compared with boor |
|-----------|-----:|------|--------------------|
| `~/src/ambulant` | 4.0 GB | hg | not yet: **hg isn't installed on sap** |
| `~/src/ambulant-documents` | 442 MB | hg | not yet (same reason). Boor has it too |
| `~/src/ambulant-private` | 1.7 MB | hg, has a `certificates` folder | not inspected (secrets) |
| `~/src/mm` | 133 MB | GRiNS, CVS | see below |
| `~/src/mmdocuments` | 2.6 GB | CVS, **new**: not on boor as far as we know | see below |

`ambulant-sandbox` and `cmif` aren't on sap.

### GRiNS: which copy is newest?

Not answered yet, but two caveats were found:

- **Host mix-up.** Sap's `mm` was checked out from
  `oratrix.oratrix.nl:/ufs/mm/CVSPRIVATE` (module `mm/demo`). On boor,
  `mm` comes from `oratrix.oratrix.com` and `cmif` from `oratrix.oratrix.nl`.
  So sap's `mm` probably corresponds to boor's **`cmif`**, not boor's `mm`.
  The `.nl` host suggests the older of the two.
- **Branch checkout.** Sap's `mm` has the sticky tag **`TGRiNS-MacOSX`**
  (`CVS/Tag`): it's a branch checkout, not the trunk. A plain `diff -r`
  between copies can therefore mislead.

Sap's `mm` has 3,255 files in 299 directories. Every `CVS/Entries` date is
2003-03-28 (probably the checkout date), and file dates run from 1991 to
2003.

**Method for the comparison:** on each copy (sap `mm`, boor `mm`, boor
`cmif`, plus any Time Machine copies), dump a manifest of path, CVS
revision and sticky tag from all `CVS/Entries` files, then compare:

- same branch, higher revision = newer;
- files present in only one copy are listed separately;
- a file whose modification time differs from the timestamp in
  `CVS/Entries` has local edits that were never committed, which may be
  unique to that copy.

### `mmdocuments`

CVS checkout of `oratrix.oratrix.com:/ufs/mm/CVSPRIVATE/mmdocuments`. File
dates run from 2000 to 2011-03-16, just after the move to hg.

| Directory | Size | Contents |
|-----------|-----:|----------|
| `SMIL30` | 438 MB | SMIL 3.0 spec working material, per module (Timing, Layout, State, ContentControl, …), plus `Tests` |
| `SMIL21` | 8 MB | The same for SMIL 2.1 (Timing21, ExtMobile, …) |
| `interop2` | 109 MB | SMIL 2.0 interoperability test cases (2000) |
| `ambulant-tests` | 300 MB | Test documents, bug-report cases, the ambulantplayer.org web pages |
| `AnnotatorDemo`, `annotatordoc` | 1.7 GB | Annotator demo, including `pyamplugin` (Python plugin) code |
| `Daisy202_text_pdtb`, `FNB` | 31 MB | DAISY 2.02 books, including "no pin" pdtb variants |
| `NoBudgetStateDemo` | 36 MB | SMIL State demo with XForms ("formfaces") |
| `ITEA_Review` | 3 MB | Review demo, including Nokia material |
| `sandbox` | tiny | A blogger experiment |
| `ambulant-private` | ? | **Not inspected** (presumably secrets, like the hg repo of that name) |

**Relevance to D4:** `ambulant-tests`, the DAISY books and `SMIL30/Tests`
are useful test material. The DAISY books are exactly the accessibility
use case. `interop2` and `SMIL30/Tests` may complement the W3C test suite.
None of it should be published before checking for third-party content and
the `ambulant-private` subdirectory.
