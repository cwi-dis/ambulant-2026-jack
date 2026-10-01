# Ambulant history — Jack's corrections and additions

Corrections and additions to [history-cold.md](history-cold.md), from the
history session with Jack (started 2026-10-01). The cold file stays unchanged,
so the difference between the two remains visible.

Each item has **Jack:** (what Jack said, lightly edited) and, where the repo
can say something about it, **Repo:** (Claude's cross-check against the git
history afterwards).

## People (question 7)

### Dick Bulterman

**Jack:** Group leader during the whole Ambulant era, and before that in the
GRiNS era. Has since retired. Important, even though he hardly committed.

**Repo:** 8 commits (2004–2005), all demo content (`Extras/Welcome`,
`Extras/DemoPresentation`) and release icons. Also mentioned in messages,
e.g. "Dick wants it for demo purposes" (2004). The cold pass mentioned him
only in passing and missed his role completely: **a leader with few commits
is invisible in git statistics.**

### Sjoerd Mullender

**Jack:** Jack and Sjoerd were the initial main implementors of GRiNS. Sjoerd
may have done a little for Ambulant early on, but had moved to a different
group by then.

**Repo:** 3 commits of his own (2003-07 CR/LF conversion; 2004-05 Wise
installers for Win32 and the handheld version). His influence shows up
elsewhere too: the first commit is notes from "the first session with
Sjoerd" (2003-04-28); Kleanthis follows a suggestion from Sjoerd (2004-04);
"careful reading of the SMIL spec (by me and Sjoerd)" (2006-10); and the
**SMIL 3.0 DTDs were written by Sjoerd** (Kees, 2008-06). So he stayed
involved as an advisor on the SMIL spec well after he moved groups.

### Kleanthis Kleanthous (= `anadelt`)

**Jack:** `anadelt` is Kleanthis Kleanthous. Heavily involved in the C++
design for roughly the first two years, possibly also Windows-specific work.
Untraceable since then; no GitHub username found.

**Repo:** Consistent: `anadelt` stops on 2003-10-15 and
`Kleanthis Kleanthous <kleanthis@sf.net>` starts on 2003-10-22. Last commit
2004-07-20, so about 15 months in total, 602 commits. Topics include the
scheduler and transitions, and the `playable_notification` interface.

**Side finding:** *all* the short CVS identities (`jack`, `anadelt`,
`dbenden`, `kees`) stop around September–October 2003 and the
`@sf.net`/`@cwi.nl` identities start then. This suggests the **move from a
CWI-internal CVS server to SourceForge CVS in October 2003**, which isn't
recorded anywhere else. *(guess, to confirm with Jack)*

### Daniel Benden and Kees Blom

**Jack:** Developers in the DIS group, involved for long stretches.

**Repo:** Daniel: Nov 2003 – Jan 2011 (828 commits, also `dbenden`, and
`koelimoe@sf.net` as email). Kees: Oct 2003 – Jul 2015 (~1,885 commits under
four identities). Kees made the last substantive commits (2015) apart from
Jack's final one in 2016.

### Pablo Cesar

**Jack:** Arrived in the group around 2008, started as a postdoc on Ambulant,
working (in Jack's memory) on maybe the Qt GUI. Later became group leader,
taking over from Dick.

**Repo, which differs on two points:** his 30 commits are from
**2006-01-27 to 2006-09-18**, and they're about **GTK, not Qt**. The first is
"Merging exp-pablo-gtk branch: Initial implementation of GTK support". Files
touched: `gui/gtk`, `player_gtk`, `include/ambulant/gui/gtk`. So Pablo wrote
the GTK back-end, and he was at CWI by early 2006. (He may of course have done
Qt work that went in under someone else's name; Kees committed the Qt pan/zoom
work in 2008.)

### Marisa DeMeglio

**Jack:** External close collaborator from the DAISY Consortium, who used
Ambulant as the basis for a DAISY book reader.

**Jack (later):** Marisa's `amis-release-30` hg branch is that DAISY book
reader, called **AMIS**.

**Repo:** 12 commits (2005-06 to 2008-10), small core fixes in `smil2`,
`common` and `net`. One mentions "it was making AMIS crash", so the reader was
most likely **AMIS** (the DAISY Consortium's reader *(knowledge)*). Several
Jack commits from 2006–2007 mention Daisy books / AMIS as the trigger.

## More people the repo shows (not yet discussed with Jack)

| Who | When | Commits | What *(repo)* |
|-----|------|--------:|---------------|
| Bo Gao (`bogao`, `Administrator@BOGAOF017`) | 2007-08 – 2012-07 | ~69 | `net`, `lib`, `smil2`: RTSP via ffmpeg, libxdispatch/GCD event processor, Windows timing fixes, seamless playback. Jack in 2014: "Bo removed a need_redraw() (in Jan-2012) which basically killed fill=remove." |
| Ishan Vaishnavi (`ishanvaishnavi`) | 2006-06 – 2006-08 | 12 | RTSP MPEG-4/MPEG-2 video, frame rate and jitter handling in `net`. Kees later refined things "based on Ishan's comments" (2008) |
| Rodrigo Laiola | 2007-11 | 7 | Python embedding: `pyambulant/player_pygtk`, `player_pyqt` |
| Tim Stevens (`tim.s.stevens@bt.com`) | 2009-05 – 2011-12 | ~7 | **BT** (British Telecom): Direct2D/DirectShow, VCE / RTP video viewer on Windows. Confirms the BT collaboration guessed in the cold pass |
| Shahab | 2011-08 – 2012-05 | 4 | Windows / VS2010 project fixes |
| Tobias | 2004 | 0 | designed the release icon ("Tobias' new icon", per Dick) |

Identities not yet mapped: `uid33605` (2010-06, 1 commit), `Administrator@JACKJANSEN9172`
(presumably Jack on a Windows machine), `jack@moes-win7`, `jack@beignet-ubuntu`.

**Jack (follow-up):** Bo, Ishan, Rodrigo and Shahab were PhD students who
used Ambulant for their own research. Tim Stevens was an external
collaborator from BT; BT (Tim and a few others) were partners in several EU
projects that used Ambulant. Tobias was CWI's graphic designer.

## Version control moves (follow-up to question 3)

**Jack:** The move from internal CWI CVS to SourceForge in autumn 2003 is
probably right.

## EU projects

**Jack:** Ambulant was used in a number of EU projects, probably including
**Ta2**, **Vconect** and **2-immerse**. BT was a partner in these.

**Repo:** what the repository shows for each one. Project names hardly ever
appear in commit messages, so most of this is inferred from timing and
features. Claude's general knowledge is marked *(knowledge)*.

### Before Ta2: BT (2008)

- 2008-09: Kees builds four "Version for BT" Firefox-plugin (npambulant)
  releases, adapted to Gecko SDK 1.9 and Firefox 2/3.
- So the BT collaboration started at least a year before Ta2 shows up in the
  commits, with the browser plugin as the deliverable. (Ta2 was an FP7
  project starting around 2008 *(knowledge)*, so this may already be early
  Ta2 work.)

### Ta2 (explicitly named 2009-09 to 2010-03)

- 2009-09-19: "enable the DirectX simple video on the desktop as well (for
  Ta2 testing)".
- 2010-01-30: merging changes from the 2.2 release branch, "I need some of
  them for Ta2".
- 2010-03-15: `WITHOUT_DIALOGS` "(which it is for Ta2)": no message boxes,
  full screen on startup. So Ambulant ran as an **unattended, embedded
  Windows component**.
- **The "VCE"**, 2009-11 to 2011-04, probably Ta2 *(guess)*: "mods to make
  VCE work with RTP video viewer" (committed by Tim, written by Jack);
  Direct2D renderer work "in the VCE"; "opening multiple documents through
  xmlrpc (in the VCE)". So the VCE was a Windows system in which Ambulant
  was remote-controlled over XML-RPC and showed live RTP video. Most of
  the Direct2D (`gui/d2`) back-end probably exists because of this.
- **Seamless playback**, 2011–2012 (Jack; Bo Gao "for MyVideos"):
  playable reuse so that consecutive clips play without gaps. *MyVideos*
  was, as far as I know, a Ta2 application for automatically edited
  personal video *(knowledge, uncertain)*, which fits with Bo's and
  Rodrigo's PhD topics.

### Between Ta2 and Vconect, or early Vconect (2012–2013)

Not attributed to a project in the repo, but clearly project-driven:

- 2012-01: `WITH_REMOTE_SYNC` / `timer_sync` API and the timesync plugin
  (Jack): synchronising Ambulant's clock with an external clock, i.e.
  synchronised playback on multiple devices. "In the way of Shahab's
  experiments" (2012-05).
- 2012-06 → 2014-11: **recorder API** (Kees) and its sandbox satellites:
  `ambulant-recorder-plugin`, a GStreamer RTSP server for "Ambulant screen
  data", `gstambulantsrc` (0.10, then 1.0). Ambulant's rendered output
  becomes a video stream.
- 2013-05: `sandbox/ambulant-server`, "Using Ambulant without video card":
  render on a headless Linux box with a dummy X server, and stream the
  result elsewhere. **Ambulant as a server-side video composer.**
- 2013-09 → 2014-01: video latency measurement (`AMBULANT_LOGFILE_LATENCY`,
  `latency2csv.py`).

### Vconect (explicitly named 2014-01 to 2014-08)

- 2014-01-20: latency logging disabled because it "destabilizes things for
  Vconect, because computing the URLs there is expensive (due to use of
  state)", so Vconect documents relied on **SMIL State**.
- 2014-02-18: RTP reorder queue size configurable "for vconect", and
  `ffmpeg_common.cpp` still has comments "Trying to get Vconect streams
  working". So Vconect used **live RTP video streams**.
- 2014-07: node restart timing fixed (seen in Vconect); 2014-08: crash when
  "switching between multiple cameras" fixed (Kees).
- So in Vconect, Ambulant was a live **video-communication composer**:
  SMIL + State choosing between multiple live camera streams. The
  2012–2013 recorder/streaming work most likely belongs here too *(guess)*.

### 2-immerse

- **No trace at all in the repo.** 2-immerse ran roughly 2015–2018
  *(knowledge)*, after Ambulant development had effectively stopped (148
  commits in 2015, mostly packaging). If Ambulant was used there, it
  wasn't developed further for it here, or that work lives somewhere else.

### Questions for Jack

1. Was the 2008 "Version for BT" already Ta2, or an earlier BT
   collaboration?
2. What did **VCE** stand for, and was it Ta2?
3. Was **MyVideos** Ta2? And the timesync / remote sync work (2012): which
   project, and was it Shahab's PhD work?
4. Was the recorder / "Ambulant as server" work (2012–2013) for Vconect?
5. 2-immerse: was Ambulant actually used there, or was it just the
   successor project that took the group away from Ambulant (to web
   technology)?
6. Were there other EU (or national, e.g. ITEA) projects before 2008? The
   early years have Nokia (770/Maemo, "Nokia tests"), Zaurus/PDA and
   WinCE targets, which suggests mobile-oriented funding.

### Answers (Jack)

- **VCE** = (probably) **Visual Composition Engine**: essentially a headless
  Ambulant player that produced "normal" video streams.
  **Repo:** fits. Besides the XML-RPC control and Python fixes "for the
  VCE", there's a run of 2011 commits making `get_screenshot()` work in the
  Direct2D back-end by rendering to a temporary software render target
  (Jack, 2011-04 → 07), plus render-target dumping (Kees, 2011-03) and
  locking the D2D bitmap (Tim, 2011-09). That's how frames got out of
  Ambulant into a video stream on Windows. The later Linux work (recorder
  API, `gstambulantsrc`, `ambulant-server` with a dummy X server,
  2012–2014) looks like the same idea, done again on Linux with SDL2 and
  GStreamer.
- **2-immerse** was probably post-Ambulant.
- **Pre-2008 projects:** Ambulant was possibly used in **iNEM4U**, **SPICE**,
  **Passepartout** or **Bricks**, but they weren't central to it. *(Repo:
  none of these names appear in commits or files.)*
- **The Nokia 770, Zaurus and WinCE ports** and a very early iOS port
  attempt (before there was an official SDK/toolchain) are not too
  important.
  **Repo:** the iOS attempt is visible. Branch `exp-jack-uikit` was merged
  on 2008-01-18, with "tweaks to make video more palatable on the iPhone"
  on 2008-01-29. Apple's iPhone SDK was only announced in March 2008
  *(knowledge)*, so this was indeed pre-SDK. The official-SDK
  `player_iphone` work comes later (2010).

## The old repositories (Jack's local checkouts, examined 2026-10-01)

**Jack:** The old Mercurial checkouts still exist on Jack's machine:
`~/src/ambulant`, `~/src/ambulant-documents`, `~/src/ambulant-sandbox`,
`~/src/ambulant-private`. `ambulant-private` contains installers and such,
and is private because it may contain secrets (certificates, signing keys).

**Findings** (with `hg` 7.2.4, read-only):

| Repo | VCS | Remote | Changesets | Span | Notes |
|------|-----|--------|-----------:|------|-------|
| `~/src/ambulant` | hg | `ssh://hg@ambulantplayer.org/hg/ambulant` | **9,162** | 2003-04-28 → 2016-07-13 | 160 named branches (158 closed), 43 tags; last pull Aug 2017 |
| `~/src/ambulant-documents` | hg | `ssh://hg@ambulantplayer.org/hg/ambulant-documents` | 149 | 2004-02-03 → 2011-04-06 | first commit: "Moving demo documents out of the source tree". Authors: Jack 64, **Dick 60**, Daniel 12, Marisa 5, … |
| `~/src/ambulant-private` | hg | `ssh://hg@ambulantplayer.org/hgpriv/ambulant-private` | 90 | 2006-04-30 → 2019-06-19 | top-level: `certificates`, `daisy-pdtb-spec`, `devices`, `fnb-installer`, `pdtbplugin`, `pdtbtests`. 24 uncommitted files in the working copy. Contents not inspected |
| `~/src/ambulant-sandbox` | **CVS**, never converted | SourceForge CVS, module `sandbox` | — | 2005–2009 | the SourceForge CVS service is gone (pserver and rsync refused), so this checkout is the only copy we know of |

### What exactly the git conversion lost

**Verified:** the GitHub repo is exactly the hg changesets that are ancestors
of the `default` tip: 9,162 − 1,987 = 7,175 = the git commit count. The
1,987 changesets that aren't ancestors of `default`:

- **154 branch-closing commits**: e.g. the 2011-03-14 commits in which Jack
  closed all the CVS-era branches after the hg conversion, and later
  single-commit closes.
- **~1,700 real commits on branches**, mainly:
  - **Release branches** (≈ 490 commits): `release-ambulant-M`, `-S`,
    `-140`, `-16-branch`, `-18-branch`, `new-18-branch`, `-20-branch` (72),
    `-22-branch` (225), and the 2.4/2.6 branches. Bug fixes were merged
    into the trunk, but **the exact code that was released isn't in git.**
  - **CVS-era feature branches** (2003–2011): CVS "merges" became flattened
    single commits on trunk, so the branch-side history steps were never
    ancestors. The *result* is in git, the step-by-step history isn't.
    The biggest: `exp-jack-keep-renderers` (160, Bo+Jack),
    `exp-bo-ffmpeg-windows` (62), `exp-pablo-gtk` (53, Pablo's GTK work),
    `exp-jack-playerobj` (46), `daniel_experimental_rtsp_datasource` (42).
  - **Never merged at all** (content not in git): `amis-release-30`
    (Marisa, 2008–2009, AMIS 3.0 release incl. audio slowdown),
    `exp-kees-plugin-ncl` (an **NCL plugin**, 2012), `exp-kees-olpc`
    (`player_olpc`, 2011), `exp-kees-sdl2-mac` (2014),
    `exp-kees-redraw` (2014), `exp-kees-gtk-video`, `exp-kees-pkttrace`,
    `exp-kees-nightlybuild`, `exp-bo-comparation-vlc-dash`,
    `exp-shahab-wallclock`, `exp-jack-raspberrypi` and others.
  - **4 orphaned changesets on `default`** (Kees, 2015-01-24..26, revs
    9109–9114). This is the faulty-merge-9087 episode: Kees redid a lost fix
    ("Fix 9107 redone (lost by merge ?). Fixes #876") and then closed that
    head "because of irrecoverable merge errors". Jack's undo on
    2015-01-21 is on the surviving line.

### Tags

43 hg tags. **Only 25 point to changesets that are in git.** The 18 that
aren't:

| Tag | hg rev | Date | On branch |
|-----|------:|------|-----------|
| `release-ambulant-141-tag` | 2279 | 2005-05-28 | release-ambulant-140 |
| `release-ambulant-144-tag` | 2363 | 2005-06-24 | release-ambulant-140 |
| `release-ambulant-145-tag`, `-145-merge` | 2449 | 2005-07-18 | release-ambulant-140 |
| `release-ambulant-16` | 2758 | 2005-12-12 | release-ambulant-16-branch |
| `release-ambulant-161`, `-16-merge` | 2813/2814 | 2006-01-05 | release-ambulant-16-branch |
| `release-ambulant-18-merge`, `-18-merge-branch-1` | 3304 | 2006-09-15 | release-ambulant-18-branch |
| `release-new-ambulant-18` (+2 merge aliases) | 3503 | 2007-02-23 | release-new-ambulant-18-branch |
| `release-ambulant-201` | 4735 | 2008-12-18 | release-ambulant-20-branch |
| `release-ambulant-202`, `-20-merge` | 4874/4873 | 2009-04-21 | release-ambulant-20-branch |
| `release-ambulant-22-merge` | 5714 | 2010-06-09 | release-ambulant-22-branch |
| `before_localcoords_merge` | 246 | 2003-09-26 | experimental_jack_localcoords |
| `exp-jack-cg-topleft-working` | 6159 | 2011-01-07 | exp-jack-cg-topleft |

In git: `initial`, `AMBULANT_0`, `release_0_0`, `release-ambulant-X`,
`-O`, `-100`, `-120`, `release-ambulant-24`, the `-merge-trunk` markers and
the `before-*` markers. **2.2 and 2.6 were never tagged** (2.6 does have
its `release-ambulant-26-branch`).

So restoring tags to GitHub is only half the job: for most releases, the
tagged commit itself has to be brought over too.

### Other copies

- **SourceForge hosts hg mirrors** (`hg.code.sf.net/p/ambulant/code` and
  `/ambulant-documents`):
  - `code` stops around **May 2014**: Jack's local clone has 536 changesets
    it lacks and the mirror has nothing Jack lacks.
  - `ambulant-documents` on SF has **1 changeset Jack's clone lacks**
    (2011-09-19, Kees: "Added a test for Video transitions (all basic
    types)").
  - **Jack's local `~/src/ambulant` is the most complete copy of the history
    we know of.**
- `ambulantplayer.org` still resolves (DigitalOcean IP) and still serves the
  old static website. hgweb returns 404. hg-over-ssh not tested.

### Issue tracker

**Correction of an earlier guess in this file:** there was no second
tracker. There is **one** tracker, SourceForge `p/ambulant/bugs`, with
**887 tickets** (#1 … #887). The old 7-digit SF IDs in early commit messages
(e.g. #2908167) are the same tickets before SourceForge's renumbering. The
3-digit ones (#838 "Animation bug in Vconect app", #876, #881 …) are the
current numbers. Everything is downloadable through SourceForge's REST API
(`https://sourceforge.net/rest/p/ambulant/bugs/`). The project also has
`support`, `reviews` and `mailman` tools, not yet looked at.

(Some other trackers are mentioned in commits: `trac.ffmpeg.org/ticket/2659`
and the DAISY consortium's AMIS Trac, `daisy-trac.cvsdude.com/amis/ticket/139`.)

## GRiNS sources (for later)

**Jack:** GRiNS (the predecessor, implemented initially by Jack and Sjoerd)
is checked out twice in Jack's `~/src`: as `mm` and as a later checkout,
`cmif` (probably without changes).

**Check:** both are **CVS working copies only** (no history locally), from
`oratrix.oratrix.com:/ufs/mm/CVSPRIVATE` (`mm`) and
`oratrix.oratrix.nl:/ufs/mm/CVSPRIVATE` (`cmif`), i.e. Oratrix's private CVS
server. `cmif` contains some extra editor scratch files (`#…#`, `@x`); real
differences not yet compared. Whether the CVS repository itself (with
history) survives somewhere is an open question.
