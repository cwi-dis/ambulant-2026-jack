# claude-notes

Working notes from the 2026 Ambulant revival experiment (Jack's fork,
`cwi-dis/ambulant-2026-jack`). Two forks are investigated in parallel — one
by Jack+Claude, one by Karthik+Claude — and compared after about a week.

## Method

1. **Cold pass** (Claude alone, no input from Jack): orient in the code,
   reconstruct history from the repository only, audit what has rotted.
   This is meant to be roughly comparable with what a newcomer+Claude
   would produce.
2. **History session with Jack**: Jack corrects and fills in the cold-pass
   history. Corrections are recorded separately, so the delta between the
   cold pass and Jack's knowledge is itself a result of the experiment.
3. Later: build reconnaissance, revival assessment.

Each phase is committed separately, so `git log -- claude-notes` shows what
was known at which point.

## Files

| File | Phase | Contents |
|------|-------|----------|
| [architecture.md](architecture.md) | cold | What the code is: layers, core objects, playback flow, platforms |
| [history-cold.md](history-cold.md) | cold | History as far as it can be reconstructed from the repo alone, plus questions for Jack |
| [rot-audit.md](rot-audit.md) | cold | External dependencies and platform APIs, and what has happened to them since 2016 |
| [history-jack.md](history-jack.md) | with Jack | Jack's corrections and additions to the cold history, cross-checked against the repo |
| [history-recovery.md](history-recovery.md) | with Jack | Deferred: where the lost hg history and the issue tracker live, and how to import them later |

## Effort

From the Claude Code session transcript (session `a5fe2aad`, 2026-10-01)
and the commit timestamps. *Claude busy* = time from Jack's message to the
end of Claude's response, summed. The rest of the wall-clock time is mostly
Jack (reading, thinking, answering, installing tools), plus any interruptions.

| Phase | Wall clock | Claude busy | Jack's turns | Commit |
|-------|-----------:|------------:|-------------:|--------|
| 0. Planning (the "grand plan") | 12:36 → 13:08, ~32 min | 0.7 min | 1 | — |
| 1. Cold pass | 13:08 → 13:13, ~5 min | 4.5 min | 1 | `33b6ace8e` 13:12 |
| 2. History with Jack | 13:31 → 14:41, ~70 min | ~8 min | 8 | `96fd3b5c0` 14:40 |

The cold pass was fast because it was all mechanical reading and grepping
(about 100 tool calls). Phase 2 was dominated by Jack's time; Claude's share
was mostly cross-checking Jack's answers against git and hg history.

## Cold pass provenance

- Date: 2026-10-01
- Model: Claude Opus 5.5 (Claude Code)
- Inputs: the repository working tree and its git history only. No web
  lookups, no input from Jack beyond "this was the group's central
  software 2003–2016". Nothing was built or run.
- Statements are marked where it matters: *(code)* = verified in source,
  *(docs)* = from in-tree documentation (which may be outdated),
  *(guess)* = inference, *(knowledge)* = Claude's general knowledge of the
  outside world (e.g. what happened to ffmpeg or NPAPI after 2016), not
  freshly verified.
