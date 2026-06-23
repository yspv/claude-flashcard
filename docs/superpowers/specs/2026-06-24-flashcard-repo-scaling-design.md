# Flashcard repo scaling design

**Date:** 2026-06-24
**Status:** Approved, pending implementation plan

## Context

`claude-flashcard` currently has one skill, `lang-flashcard`, plus a
`CLAUDE.md` describing conventions for adding more domain-specific flashcard
skills (medical, coding, exam-prep, etc.). The user is planning to add
flashcard skills across many domains, not just one more. Two parts of the
current convention don't scale past 2-3 skills:

1. **Collision avoidance is pairwise.** `lang-flashcard`'s description says
   to prefer itself "over the general flashcard skill" — an N² pattern where
   every skill must know about and out-rank every other skill's description.
2. **Domain-agnostic content is duplicated inline.** The CSV output format
   (header, escaping rules, summary footer) and the generic portion of the
   SuperMemo Twenty Rules checklist are fully written out inside
   `lang-flashcard/SKILL.md` and would otherwise be copy-pasted into every
   new domain skill verbatim.

This design resolves both before the second domain skill is written.

## Goals

- Adding a new domain skill should require zero edits to any other skill or
  to a routing list.
- Domain-agnostic output format and quality rules live in one place; domain
  skills extend them rather than restating them.
- Ambiguous/generic flashcard requests (no domain stated or inferable) are
  handled gracefully instead of relying on every skill description to
  out-rank every other one.

## Non-goals

- Building the actual next domain skill (medical/coding/exam-prep) — this
  spec only changes the repo-level scaffolding and `lang-flashcard`'s
  pointers into it.
- Enforcing a universal `type` taxonomy across domains — `type` stays
  free-text, defined per-skill.

## Directory structure

```
skills/
  flashcard-router/
    SKILL.md
  _shared/
    csv-output-format.md
    quality-rules.md
  lang-flashcard/
    SKILL.md
  <future-domain-skill>/
    SKILL.md
```

`_shared/` has no `SKILL.md`, so Claude Code's skill loader does not treat it
as an invokable skill — it's reference material only, linked to from domain
skills.

## `flashcard-router` skill

**Purpose:** catch generic, domain-ambiguous flashcard requests that don't
match any domain skill's own trigger vocabulary (e.g. "make me some
flashcards from this," with no subject/domain stated or inferable from the
content). In the normal case, domain skills claim their own specific
territory and fire directly — the router is the fallback path, not the
common path.

**Frontmatter description:** states plainly that this is the fallback for
flashcard requests with no identifiable domain, and that domain-specific
skills (language, medical, coding, exam-prep, etc.) should be preferred
whenever their domain is stated or inferable from the input.

**Behavior:**
1. Inspect the request and any provided material for domain signals.
2. **Discover available domain skills dynamically** by reading the
   `skills/` directory at invocation time (excluding `_shared/` and itself).
   Do not maintain a hardcoded list of domains/skills inside this file —
   that list would need a manual update every time a domain skill is added,
   which defeats the purpose of the router.
3. If exactly one discovered domain skill is a confident match, say which
   skill it's handing off to and invoke it — no need to ask the user
   anything further.
4. If multiple domain skills could plausibly match, or none clearly do, ask
   the user one question: "What subject/domain are these flashcards for?"
   with examples drawn from the currently discovered domain skills, then
   route based on the answer.
5. If the user names a domain with no matching skill yet, say so plainly
   and offer to proceed with a generic CSV flashcard format (per
   `_shared/csv-output-format.md`) instead of silently failing or guessing.

## `_shared/csv-output-format.md`

Generalizes the existing "Output format" section from `lang-flashcard`.
Contents:

- Output as a `.csv` artifact (no plain numbered lists, no HTML).
- Header row: `id,type,topic,front,back,notes`.
- Column definitions:
  - `id` — sequential integer starting at 1.
  - `type` — **free-text, defined by the invoking domain skill**; this file
    does not enumerate values. Each domain skill documents its own set
    (e.g. `lang-flashcard` defines `vocab`, `production`, `grammar`,
    `collocation`, `false-friend`, `dialogue`, `culture`, `pronunciation`,
    `mnemonic`).
  - `topic` — domain-specific subject area string (e.g. `Spanish / travel`,
    `Cardiology / pharmacology`).
  - `front` — question/prompt side; `[...]` for cloze gaps.
  - `back` — answer side.
  - `notes` — optional metadata tag; empty string if none.
- CSV formatting rules: comma-separated, UTF-8, quote fields containing
  commas/newlines/quotes, double internal quotes, real newlines (not
  literal `\n`) inside multi-line quoted fields, no extra blank lines
  between rows.
- Worked example (kept generic/illustrative, not language-specific).
- Chat summary footer format (plain text, after the artifact): total cards
  and breakdown by type, any items skipped and why, any flags for unclear
  source material, reminder to shuffle/interleave during review.

## `_shared/quality-rules.md`

Generalizes the domain-agnostic subset of the SuperMemo Twenty Rules
checklist currently in `lang-flashcard`'s Step 4. Contents:

- One fact only per card — split multi-part questions.
- Answer as short as possible — trim unnecessary words.
- Noun-based prompts where possible over full interrogative sentences.
- Difficulty matched to the learner's stated or inferred level.
- No bare term → definition pairs unless the user explicitly asks.
- Disambiguation cue added when two cards could be confused.

Domain-specific checklist items (e.g. `lang-flashcard`'s production-pairing
rule, false-friend flagging, context-card-per-vocab-item rule) stay inside
each domain skill's own `SKILL.md`, written as additions to this shared
list, not replacements for it.

## Changes to `lang-flashcard/SKILL.md`

- **Frontmatter `description`:** remove the "prefer this skill over the
  general flashcard skill" clause (no longer needed now that the router
  handles disambiguation); sharpen the description to focus on its own
  trigger vocabulary (target language names, language-learning terminology).
- **Output format section:** replace the fully inlined CSV rules with a
  pointer to `../_shared/csv-output-format.md`, keeping inline only the
  `type` enum specific to this skill and its worked example.
- **Step 4 quality checklist:** keep the language-specific bullets
  (context card per vocab item, production pairing, false friends flagged,
  disambiguation), add a pointer line to `../_shared/quality-rules.md` for
  the generic items, and remove the bullets now covered there (one fact
  only, short answers, noun-based prompts, difficulty matched to level —
  this last one keeps its existing level-specific elaboration in Step 2.5,
  only the generic checklist bullet itself moves).

## Changes to `CLAUDE.md`

- **Structure section:** document `flashcard-router/` and `_shared/`
  alongside the per-skill directory pattern.
- **Current skills section:** list `flashcard-router` separately as
  infrastructure (not a domain skill), distinct from the domain skill list.
- **Adding a new flashcard skill steps:** replace the current step 4
  (pairwise collision wording) with: reference
  `_shared/csv-output-format.md` and `_shared/quality-rules.md` from the new
  skill instead of restating them; no router updates are needed since
  domain discovery is dynamic.
- **Output convention section:** point to `_shared/csv-output-format.md` as
  the source of truth instead of inlining the schema description.

## Testing / validation

This is a documentation/skill-instruction change, not application code —
there is no automated test suite. Validation is manual:
- Read through `flashcard-router/SKILL.md` and confirm the dynamic
  discovery instructions are unambiguous about where to look (`skills/`,
  excluding `_shared/` and itself) and what to do when zero or one domain
  matches.
- Read through the updated `lang-flashcard/SKILL.md` end-to-end and confirm
  it still fully specifies card generation without the reader needing to
  guess at anything that moved to `_shared/`.
- Confirm `CLAUDE.md`'s "Adding a new flashcard skill" steps, followed
  literally, would produce a skill consistent with this design.
