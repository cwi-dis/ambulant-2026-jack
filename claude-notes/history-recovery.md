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
