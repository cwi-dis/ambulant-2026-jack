# Ambulant history — cold reconstruction

Reconstructed from the git history and in-tree files only, without Jack's
input. To be corrected in the history session.

## The repository's own history

The code has moved between version control systems at least twice, and every
move lost something:

1. **CVS on SourceForge**, 2003 → Feb 2011. *(docs: organisation.txt,
   README-Maintenance)* For a while in 2009 some commits went through a
   CVSNT server (the "Committed on the Free edition of March Hare Software
   CVSNT Server" spam in Kees's commit messages).
2. **Mercurial on ambulantplayer.org**, Feb 2011 → 2016. Converted with
   cvs2svn's `cvs2hg` (`scripts/cvs2hg-ambulant.options`, 2011-02-11). The
   conversion turned CVS tags into hg tags, and left "cvs2hg fixup commit for
   tag ..." commits as traces. *(code)* There were also two sibling hg repos,
   `ambulant-documents` (test/demo documents) and `ambulant-private`.
   *(docs: README-hg)*
3. **Git on GitHub** (`cwi-dis/ambulant`), date and tool unknown. This fork
   (`cwi-dis/ambulant-2026-jack`) is identical to upstream (0 commits
   ahead or behind).

What the git conversion lost *(code)*:

- **All tags.** There are 0 git tags and no `.hgtags` file, although the
  history shows tags such as `release-ambulant-18-merge-trunk-1`,
  `release-ambulant-141-tag`, `release-ambulant-22-merge-trunk`,
  `AMBULANT_0`.
- **Named branches.** hg used named branches (`release-ambulant-NN-branch`,
  `exp-jack-*`, `exp-kees-*`). Git has only `master`. The 327 merge
  commits survive, but there's no branch label on either side.
- No `.gitignore`. `.cvsignore`, `.hgignore` and `.hgeol` are still there,
  so nobody has worked in this repo since it became git.

Other scars:

- 2015-01-21, Jack: *"Undoing faulty merge 9087 which threw away half the
  changes. Will have to redo 9088. Also loses history of 9062-9086."* So
  history in that range is known to be incomplete.
- Duplicate identities: `jack`/`Jack Jansen`, `kees`/`Kees Blom`,
  `dbenden`/`Daniel Benden`. Probably CVS usernames that were only partly
  mapped to full names. `anadelt` (125 commits, 2003 only) wasn't mapped.

## Releases (from commit messages; no tags to confirm)

| Version | Approx. date | Evidence |
|---------|--------------|----------|
| 0.x ("S" release, "O" release) | early–mid 2004 | "S-release version", "New Release 'O' icon" |
| 1.0 | July 2004 | "Updated text for 1.0 release" |
| 1.2 | Dec 2004 | "Last Zaurus related changes for the 1.2 release" |
| 1.4 (–1.4.5) | Apr–Jul 2005 | "Updated for release 1.4", merges of 1.4.x branch |
| 1.6 (1.6.1) | Dec 2005–Jan 2006 | "release-ambulant-16-branch" |
| 1.8 | Sep 2006–Feb 2007 | "Update preparing for 1.8 release" |
| 2.0 (2.0.1, 2.0.2) | Dec 2008–Apr 2009 | "final mods for 2.0 distribution" |
| 2.2 | Dec 2009 | "Updated version numbers to 2.2" |
| 2.3 nightly | Dec 2010 | "Version for 2.3.20101221 distribution" |
| 2.4 | Dec 2012 | "Getting ready for 2.4" |
| 2.6 | Jan–Feb 2015 | "Upped version numbers to 2.6" |
| 2.7 | never released | trunk after Feb 2015 |

Odd-numbered versions look like development versions (1.7, 2.3, 2.5, 2.7).
*(guess)*

## Eras (commit activity)

Commits per year: 2003 505 · 2004 1227 · 2005 497 · 2006 413 · 2007 530 ·
2008 465 · 2009 395 · 2010 428 · 2011 949 · 2012 676 · 2013 459 · 2014 482 ·
2015 148 · 2016 1.

**2003–2004: design and initial build.** The first commit (2003-04-28) is a
design-notes file containing "the results of the first session with Sjoerd".
The 2003 team: jack, anadelt, Kleanthis Kleanthous (Windows / WinCE, it
seems), Daniel Benden, Kees Blom from 2004. Dick Bulterman appears
occasionally. Original targets included Windows CE (`player_wince`) and
Linux PDAs (Zaurus, later Nokia 770/Maemo), plus Qt on Linux. The first
design notes refer to **GRiNS** as the predecessor whose code organisation was
"good enough". *(docs: remarks.txt)* The peak was in 2004, the year of 1.0.

**2005–2010: SMIL 2.1 / 3.0, steady state.** About 400–500 commits a year by
a stable three-person core (Jack, Kees, Daniel). SMIL 2.1 support (2005),
SMIL 3.0 / smilText / SMIL State. Browser plugins: the Firefox/NPAPI plugin
(2006–2008; "Version for BT" in 2008 suggests a British Telecom deliverable)
and the IE/ActiveX plugin (2009). Platform churn: Cocoa → CoreGraphics
renderer, iOS port (`exp-jack-uikit` merged Jan 2008), DirectX → Direct2D
("preparing for D2D merge", Nov 2010). Daniel Benden's commits stop in early
2011.

**2011–2012: second peak.** The move to hg (Feb 2011) coincides with the second
peak (949 commits in 2011). Bo Gao experimented with libxdispatch / GCD
event processing (2011). Debian/Ubuntu packaging and PPAs, nightly builds,
SDL2 player. "Ta2" is mentioned in 2010 (a project name? *(guess)*). Hooks for
distributed synchronisation, recording and retransmission, and low-latency
live streaming (NEWS for 2.6). These look like requirements from
networked-media research projects. *(guess)*

**2013–2015: maintenance and wind-down.** Mostly keeping builds green against
moving platforms (Xcode, OSX deployment targets, Ubuntu releases, ffmpeg
versions from a third-party PPA). In 2014 Kees has more commits than Jack for
the first time. 2.6 is released in early 2015, described in NEWS as "mainly a
cleanup release ... probably the last release with browser plugins".

**July 2015 / July 2016: the end.** The ffmpeg PPA (`ppa:samrog131`) the
Linux builds depended on disappeared, so the nightly PPA builds were disabled.
The final commit (2016-07-13) switches to Ubuntu-supplied ffmpeg.

## Platforms that came and went *(code: deleted directories)*

| Removed | What it was |
|---------|-------------|
| `src/player_wince` (+`player_wincon`) | Windows CE / Pocket PC player |
| `src/player_unix` + `gui/qt` | Qt-based Linux player |
| `src/player_gtk/nokia770` | Nokia 770 / Maemo port |
| `gui/cocoa` | original macOS renderer, replaced by `gui/cg` |
| `gui/dx` | DirectX renderer, replaced by `gui/d2` |
| `gui/dg` | unknown (another Windows back-end?) |

## Questions for Jack

1. **Tags and branches:** Is there still a Mercurial repo somewhere (an
   ambulantplayer.org backup, a CWI file server, someone's old laptop) with
   `.hgtags` and the named branches? Restoring tags would be cheap and very
   useful.
2. **Sibling repos:** Do `ambulant-documents` and `ambulant-private` still
   exist? The test documents matter a lot for any revival, because there's
   nothing to regression-test against otherwise.
3. **Who did the git conversion, and with which tool?** That determines
   whether the tags can be recovered from the hg side.
4. **GRiNS:** what was its relationship to Ambulant (code reuse, or only
   design)? Is GRiNS itself somewhere in mm-group's future?
5. **Sjoerd and the 2003 design session.** What drove the original design
   (the WinCE/PDA targets seem to have been very important at the start)?
6. **Funding projects:** Which projects drove which features (BT, Nokia,
   Ta2, others)? That explains, for example, why the timesync, recording and
   state plugins exist.
7. **anadelt, Kleanthis, Bo Gao, Marisa DeMeglio, Pablo Cesar:** roles?
8. **The faulty merge 9087 in Jan 2015:** was the lost range 9062–9086 ever
   fully redone?
9. **What was the website ambulantplayer.org running on**, and is any of it
   (demos, FAQ, bug tracker export from SourceForge) archived?
