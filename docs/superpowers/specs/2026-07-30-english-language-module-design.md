# English language module for lang-flashcard — design

Date: 2026-07-30
Status: approved approach, spec for implementation

## Problem

`lang-flashcard` is deliberately language-agnostic: its nine card types
(A–I) work for any target language, but they carry no knowledge of what
makes a *specific* language hard. English learners struggle with a
well-known, recurring set of pain points — verb patterns (*stop to
smoke* vs *stop smoking*), near-synonyms that split by situation
(*say/tell*, *do/make*), collocational restrictions (*make a mistake*,
never *do a mistake*), phrasal verbs, articles, and a deep
spelling–pronunciation mismatch. Today the skill only covers these if
the model happens to think of them; nothing steers card generation
toward them systematically.

## Goal

Add per-language guidance modules that `lang-flashcard` loads on demand,
starting with English. The module tells the skill *what to hunt for* in
English source material and *how to slant the existing card types* at
those targets. It also tells the skill how to derive
native-language-specific interference patterns for whatever L1 the
learner states — as instructions, not hardcoded per-L1 tables.

## Chosen approach

**Reference module loaded on demand** (approach 1 of 3 considered).

- New file: `skills/lang-flashcard/references/languages/english.md`.
- `lang-flashcard/SKILL.md` gains a small step after intake: check
  `references/languages/<target-language>.md` (lowercase English name of
  the target language); if the file exists, read it fully and apply its
  guidance throughout Steps 1–4. If it doesn't exist, proceed exactly as
  today.
- No new skill, no router changes, no new CSV `type` values. The module
  only slants the existing card types A–I, so the shared output contract
  in `_shared/csv-output-format.md` is untouched.

Rejected alternatives: a sibling `english-flashcard` skill (trigger
description would overlap with `lang-flashcard`, and every future
language would duplicate the whole pipeline) and a nested SKILL.md
subskill (Claude Code has no parent–child skill relationship; it would
surface as an ambiguous independent skill).

## Module content — `references/languages/english.md`

Every section names which existing card type(s) it uses and how. The
module adds hunting instructions and slanting rules, not new machinery.

### 1. Verb patterns (gerund vs infinitive)

- Verbs taking *to*-infinitive (*want, decide, refuse*), verbs taking
  *-ing* (*enjoy, avoid, finish*), verbs taking both with **meaning
  change** (*stop, remember, forget, try, regret, go on*).
- Card guidance: use type C (grammar cloze) with the pattern as the gap.
  For meaning-change verbs, **always generate a contrast pair** — two
  cards whose sentences force the two different meanings — plus a
  disambiguation cue so the pair isn't confusable (per shared quality
  rules).

### 2. Near-synonym discrimination

- Pairs/groups with similar meaning but different situations of use:
  *say/tell*, *do/make*, *borrow/lend*, *bring/take*, *hear/listen*,
  *look/watch/see*, *travel/trip/journey*, *fun/funny* and the like.
- Card guidance: never define these words in isolation. Use a cloze
  (type A or C) where **only one of the pair fits**, with the deciding
  rule on the back (e.g. "*tell* + person, *say* + words"). Scan source
  material for members of known confusable sets and card the distinction
  even if only one member appears.

### 3. Collocation restrictions

- English-specific extension of generic type D: verb+noun traps
  (*make a mistake / do homework / take a photo*), adjective+noun
  (*heavy rain*, not *strong rain*; *strong coffee*, not *powerful
  coffee*), fixed prepositions (*depend on*, *arrive at/in*, *married
  to*).
- Card guidance: type D cloze on the restricted slot; back gives the
  correct collocate **and names the natural-sounding wrong guess** the
  learner would likely make, marked wrong.

### 4. Classic English pain points

- **Phrasal verbs** — treat each as a vocabulary item in its own right
  (type A cloze in context + type B production for high-frequency ones).
  Note separable vs inseparable on the back when it matters (*look it
  up* vs *look after it*). Prefer phrasal verbs actually present in the
  source material over generic lists.
- **Articles (a/an/the/∅)** — type C cloze with the article as the gap;
  back states the specific rule instance (first vs second mention,
  unique things, generalizations with ∅). Emphasized further by L1
  derivation (§5) when the learner's L1 lacks articles.
- **Spelling–pronunciation mismatch & word stress** — type H cards for
  silent letters (*comb, receipt, island*), heteronyms (*read/read,
  record* n./v.), and stress-shift pairs (*PREsent/preSENT*). Follow the
  existing type H convention: plain-language respelling first, IPA in
  parentheses.
- **Irregular verbs** — only card irregular forms encountered in or
  implied by the material; type C cloze in a sentence, never bare
  paradigm tables (a paradigm is multiple facts — split it).

### 5. L1 interference derivation (instructions, not tables)

The module instructs — for whatever native language the learner stated
in intake — deriving where English will interfere, then weighting card
generation toward those areas. The derivation checklist:

- Does the L1 have articles? If not → increase article-card share.
- Which English phonemes are missing from the L1 (/θ/, /ð/, /w/–/v/,
  /ɪ/–/iː/…)? → minimal-pair type H cards for those.
- Does the L1 mark verb aspect/tense differently? → target the tenses
  English splits that the L1 merges (e.g. past simple vs present
  perfect).
- Known false friends between the L1 and English → type E cards
  (existing rule, restated here as part of the derivation pass).
- Word-order and preposition calques → collocation/grammar cards for
  the specific patterns the L1 would transfer wrongly.

With **2–3 worked examples of the derivation** (e.g. "L1 = Russian:
article-less → article drills; no /θ/ → *think/sink* minimal pairs;
single past → past simple vs present perfect contrast cards") — the
examples show the *method*, they are not an exhaustive per-L1 table.

### 6. English-specific quality additions

Short checklist appended to Step 4's language-specific rules when the
module is loaded, e.g.:

- Meaning-change verb-pattern cards come in contrast pairs.
- Near-synonym cards test the *choice between* members, not definitions.
- Phrasal verbs are carded as units, with separability noted when it
  matters.
- L1 derivation pass performed when the native language is known.

## Supporting edits

1. **`skills/lang-flashcard/SKILL.md`** — add a short "Step 0.5: Load
   language-specific guidance" (placement after intake, **before**
   Step 1): check `references/languages/<language>.md`, read fully if
   present, apply throughout. It must load before the resource is read,
   because the module contains hunting instructions that shape what to
   look for in the material. A few lines only; must not restate module
   content.
2. **`CLAUDE.md`** — document the `references/languages/` pattern under
   the structure section and in "Adding a new flashcard skill" territory
   (a note that per-language modules for lang-flashcard go in
   `skills/lang-flashcard/references/languages/<language>.md`, named by
   the lowercase English name of the language).

## Out of scope

- Modules for other languages (the pattern supports them; English is the
  only deliverable now).
- New CSV `type` values or changes to `_shared/` files.
- Router changes.
- Hardcoded per-L1 interference tables.

## Success criteria

- A request like "make flashcards from this article, I'm learning
  English, native language Uzbek" loads the module and produces a deck
  that includes verb-pattern contrast pairs, near-synonym choice cards,
  collocation-restriction cards, and L1-derived emphasis (articles,
  minimal pairs) — using only existing card types and the shared CSV
  schema.
- A request for any other target language behaves exactly as before the
  change.
