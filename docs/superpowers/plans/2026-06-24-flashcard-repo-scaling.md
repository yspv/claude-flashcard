# Flashcard Repo Scaling Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure the `claude-flashcard` repo so new domain flashcard skills (medical, coding, exam-prep, etc.) can be added without editing any other skill, by introducing a dynamic-discovery router skill and a `_shared/` reference directory for the CSV output format and generic quality rules.

**Architecture:** Two new shared reference files (`skills/_shared/csv-output-format.md`, `skills/_shared/quality-rules.md`) hold domain-agnostic content. A new `flashcard-router` skill catches domain-ambiguous requests by scanning `skills/` at invocation time — no hardcoded skill list. `lang-flashcard/SKILL.md` and `CLAUDE.md` are edited to point at the shared files instead of inlining their content, and to drop the now-obsolete pairwise collision-avoidance wording.

**Tech Stack:** Markdown only (Claude Code skill files: `SKILL.md` with YAML frontmatter + Markdown body). No code, no test runner — this is an instruction/documentation change. "Testing" means a careful manual read-through of each file against an explicit checklist, since the consumer of these files is Claude Code itself, not a compiler.

## Global Constraints

- CSV output schema is fixed repo-wide: header `id,type,topic,front,back,notes` (from spec, `_shared/csv-output-format.md` section).
- `type` column is free-text, defined per-skill — no universal enum (spec Non-goals).
- `_shared/` must contain no `SKILL.md` file, so it is never loaded as an invokable skill (spec Directory structure section).
- The router must discover domain skills dynamically by reading the `skills/` directory at invocation time; it must NOT hardcode a list of domains or skills (spec `flashcard-router` Behavior, step 2).
- Adding a future domain skill must require zero edits to `flashcard-router/SKILL.md`, `lang-flashcard/SKILL.md`, or any other existing skill (spec Goals).
- Domain-specific quality-checklist items and CSV `type` values stay inside each domain skill's own `SKILL.md`; only the genuinely generic items move to `_shared/` (spec `_shared/quality-rules.md` section, last paragraph).

---

### Task 1: Create `skills/_shared/csv-output-format.md`

**Files:**
- Create: `skills/_shared/csv-output-format.md`

**Interfaces:**
- Consumes: nothing (new standalone reference file).
- Produces: a Markdown reference file that domain skills link to via the relative path `../_shared/csv-output-format.md`. Section headers inside this file (`## Header row`, `## Column definitions`, `## CSV formatting rules`, `## Example`, `## Summary footer`) are referenced by name from Task 4's edits to `lang-flashcard/SKILL.md`, so keep these exact header strings.

- [ ] **Step 1: Write the file**

Create `skills/_shared/csv-output-format.md` with this content:

```markdown
# Shared CSV output format

Used by all flashcard skills in this repo. Domain skills should link here
instead of restating these rules, and only document their own `type` enum
and a domain-specific worked example inline.

Output the cards as a **CSV artifact** — no plain numbered lists, no HTML,
no React components. Cards are study material the user will copy into
their flashcard app (Anki, Quizlet, etc.).

## Header row

```
id,type,topic,front,back,notes
```

## Column definitions

- **id** — sequential integer starting at 1.
- **type** — free-text, **defined by the invoking domain skill**. This file
  does not enumerate values; each domain skill documents its own set (e.g.
  `lang-flashcard` defines `vocab`, `production`, `grammar`, `collocation`,
  `false-friend`, `dialogue`, `culture`, `pronunciation`, `mnemonic`).
- **topic** — domain-specific subject area string (e.g. `Spanish / travel`,
  `Cardiology / pharmacology`).
- **front** — the question or prompt side of the card; use `[...]` for
  cloze gaps.
- **back** — the answer side of the card.
- **notes** — optional: register, pattern label, source tag, or other
  metadata — empty string if none.

## CSV formatting rules

- Comma-separated, UTF-8 encoded.
- Wrap any field containing commas, newlines, or double-quotes in
  double-quotes.
- Escape internal double-quotes by doubling them (`""`).
- Multi-line answer content uses a **real newline** inside the quoted
  field — do NOT use the literal escape sequence `\n`.
- Do not add extra blank lines between rows.

## Example

```csv
id,type,topic,front,back,notes
1,concept,Example domain / topic,"What is the prompt side of a card?","The question or cue the learner sees first.",""
2,application,Example domain / topic,"""embarazada"" — false friend for English speakers?","Yes — means 'pregnant' NOT 'embarrassed'.",""
3,dialogue,Example domain / topic,"Multi-line example — what does the back column support?","Real newlines inside a quoted field.
Like this second line.",""
```

## Summary footer

After the CSV artifact, add a short summary in chat (plain text):
- Total cards, breakdown by type.
- Any items skipped (and why — too listy, out of scope, etc.).
- Any flags for unclear source material.
- Reminder to shuffle / interleave cards during review.
```

- [ ] **Step 2: Verify the file reads as a complete, self-contained reference**

There is no test runner for this — verification is a manual read-through.
Open the file and confirm, reading top to bottom with no other context:
- Every section in Step 1's content is present: `## Header row`,
  `## Column definitions`, `## CSV formatting rules`, `## Example`,
  `## Summary footer`.
- The `type` column description explicitly says "free-text, defined by the
  invoking domain skill" and does NOT enumerate a fixed list of values.
- The example CSV block is syntactically valid CSV (each row has exactly 6
  comma-separated fields once quoting is accounted for) and uses a generic
  domain (not Spanish/language-specific) for rows 1 and 3, matching the
  spec's "kept generic/illustrative, not language-specific" requirement —
  row 2 may keep a language example since it's illustrating quote-escaping,
  not domain content.

Run this check to confirm the file was created and is non-empty:

```bash
wc -l skills/_shared/csv-output-format.md
```

Expected: a line count greater than 0 (around 45-55 lines).

- [ ] **Step 3: Commit**

```bash
git add skills/_shared/csv-output-format.md
git commit -m "Add shared CSV output format reference for flashcard skills"
```

---

### Task 2: Create `skills/_shared/quality-rules.md`

**Files:**
- Create: `skills/_shared/quality-rules.md`

**Interfaces:**
- Consumes: nothing (new standalone reference file).
- Produces: a Markdown reference file linked from domain skills via
  `../_shared/quality-rules.md`. The checklist items in this file must
  remain a strict subset of what's listed in the spec's
  `_shared/quality-rules.md` section so Task 4 can correctly identify which
  bullets to remove from `lang-flashcard/SKILL.md`'s own checklist.

- [ ] **Step 1: Write the file**

Create `skills/_shared/quality-rules.md` with this content:

```markdown
# Shared quality rules

Domain-agnostic subset of the SuperMemo Twenty Rules, used by all flashcard
skills in this repo. Domain skills should link here and then list their
own additional, domain-specific checklist items — these rules are a
baseline to extend, not to replace.

Before outputting any deck, check each card against:

- [ ] **One fact only** — if a question has two parts, split it into two
  cards.
- [ ] **Answer as short as possible** — trim every unnecessary word.
- [ ] **Noun-based prompts where possible** — prefer a short cue phrase
  over a full interrogative sentence when the meaning stays clear (e.g.
  "tarde — meaning" rather than "What does tarde mean?").
- [ ] **Difficulty matched to the learner's level** — a card should be one
  the learner can *almost* always get, with effort. Domain skills should
  define what "level" means for their domain (proficiency, year of study,
  seniority, etc.) and how it changes scaffolding.
- [ ] **No bare term → definition pairs** unless the user explicitly asks
  for them — prefer a card that shows the term in context or use.
- [ ] **Disambiguation cue added** when two cards in the deck could be
  confused with each other.
```

- [ ] **Step 2: Verify the file reads as a complete, self-contained reference**

Manual read-through check:
- All six checklist bullets from Step 1 are present and match the spec's
  `_shared/quality-rules.md` list (one fact only; short answers; noun-based
  prompts; difficulty matched to level; no bare term→definition pairs;
  disambiguation cue).
- No domain-specific items leaked in (no mention of "production card",
  "false friend", "vocabulary item", or any other `lang-flashcard`-specific
  term — those stay in `lang-flashcard/SKILL.md` per the spec).

Run this check to confirm the file was created and is non-empty:

```bash
wc -l skills/_shared/quality-rules.md
```

Expected: a line count greater than 0 (around 15-20 lines).

- [ ] **Step 3: Commit**

```bash
git add skills/_shared/quality-rules.md
git commit -m "Add shared quality rules reference for flashcard skills"
```

---

### Task 3: Create `skills/flashcard-router/SKILL.md`

**Files:**
- Create: `skills/flashcard-router/SKILL.md`

**Interfaces:**
- Consumes: `../_shared/csv-output-format.md` (referenced by path for the
  fallback generic-format offer, per spec Behavior step 5). Does not
  consume anything from `lang-flashcard/SKILL.md` directly — discovery is
  dynamic, not a hardcoded reference.
- Produces: a new invokable skill named `flashcard-router`. Its frontmatter
  `description` is read by Claude Code's skill-matching at request time, so
  the exact wording matters for when this skill fires relative to domain
  skills like `lang-flashcard`.

- [ ] **Step 1: Write the file**

Create `skills/flashcard-router/SKILL.md` with this content:

```markdown
---
name: flashcard-router
description: >
  Fallback router for flashcard requests where no specific domain (language,
  medical, coding, exam subject, history, science, etc.) is stated or
  inferable from the request or the provided material. Domain-specific
  flashcard skills should always be preferred over this skill whenever
  their domain is stated or can be inferred — this skill only engages for
  genuinely ambiguous requests like "make flashcards from this" with no
  identifiable subject, or when the user asks which flashcard skill to use.
  Do not use this skill if a domain skill's own description already
  matches the request.
---

# Flashcard Router Skill

Routes domain-ambiguous flashcard requests to the right domain-specific
skill, or asks the user to disambiguate. This skill is the fallback path —
in the normal case, domain skills (language, medical, coding, exam-prep,
etc.) claim their own specific territory through their own descriptions and
fire directly without this skill ever engaging.

---

## Step 1: Discover available domain skills

Before doing anything else, list the contents of the repo's `skills/`
directory to see which domain skills currently exist. Do this fresh on
every invocation — never hardcode a list of domains or skill names in this
file. The whole point of this router is that adding a new domain skill
requires zero edits here.

Exclude from consideration:
- `_shared/` — this is a reference directory, not a skill (it has no
  `SKILL.md`).
- `flashcard-router/` — this skill itself.

For each remaining directory, read its `SKILL.md` frontmatter
`description` to understand what domain and trigger phrases it covers.

## Step 2: Match the request against discovered domains

Look at the user's request and any provided material (text, URL, PDF,
image, topic name) for domain signals — subject matter, terminology,
explicit statements like "for my Spanish class" or "USMLE prep".

- **Exactly one domain skill is a confident match** — state which skill
  you're handing off to, then invoke it. Do not ask the user anything
  further.
- **Multiple domain skills could plausibly match, or none clearly do** —
  ask the user one question: "What subject or domain are these flashcards
  for?" Offer a few examples drawn from the domain skills you discovered in
  Step 1 (e.g. "language learning, medical, coding, exam prep, or
  something else?"). Route to the matching skill based on their answer.
- **The user names a domain with no matching skill yet** — say so plainly.
  Do not silently guess or invent a domain skill. Offer to proceed anyway
  using the generic CSV flashcard format described in
  `../_shared/csv-output-format.md`, generating cards from your own
  knowledge of the stated domain.

## Step 3: Hand off

When invoking a discovered domain skill, do so directly — don't re-ask the
intake questions that skill will itself ask. Your job is purely to identify
the right skill (or confirm none exists) and get out of the way.
```

- [ ] **Step 2: Verify the file reads as a complete, self-contained skill**

Manual read-through check:
- Frontmatter has valid YAML with `name: flashcard-router` and a
  `description` field.
- The description explicitly states this is a fallback and that
  domain-specific skills take precedence — re-read it and confirm a
  reasonable person would not expect this skill to fire for a request like
  "Spanish flashcards" given `lang-flashcard`'s description exists.
- Step 1 explicitly says to exclude `_shared/` and `flashcard-router/`
  itself, and explicitly says never to hardcode a skill list.
- Step 2 covers all three outcomes from the spec: confident single match,
  ambiguous/no match (ask user), and named-domain-with-no-skill (offer
  generic fallback referencing `_shared/csv-output-format.md`).
- The relative path `../_shared/csv-output-format.md` is correct given
  this file lives at `skills/flashcard-router/SKILL.md` (one level up to
  `skills/`, then into `_shared/`).

Run this check to confirm the file was created and the relative path
target exists:

```bash
test -f skills/flashcard-router/SKILL.md && echo "router exists"
test -f skills/_shared/csv-output-format.md && echo "shared csv format exists"
```

Expected output:
```
router exists
shared csv format exists
```

- [ ] **Step 3: Commit**

```bash
git add skills/flashcard-router/SKILL.md
git commit -m "Add flashcard-router skill for domain-ambiguous requests"
```

---

### Task 4: Update `skills/lang-flashcard/SKILL.md` to use shared references

**Files:**
- Modify: `skills/lang-flashcard/SKILL.md`

**Interfaces:**
- Consumes: `../_shared/csv-output-format.md` and `../_shared/quality-rules.md`
  (both created in Tasks 1-2) by relative-path reference.
- Produces: an updated `lang-flashcard/SKILL.md` that no longer contains the
  "prefer this skill over the general flashcard skill" frontmatter clause,
  no longer inlines the full CSV formatting rules, and no longer inlines
  the generic quality-checklist bullets. All other content (card types,
  intake questions, deck sizing, level adaptation, language-specific
  checklist items, `type` enum, worked example) is unchanged.

- [ ] **Step 1: Update the frontmatter `description`**

The current frontmatter (lines 1-15 of the existing file) is:

```yaml
---
name: lang-flashcard
description: >
  Generate high-quality language learning flashcards from any input, following
  the SuperMemo Twenty Rules and language acquisition research. Use this skill
  whenever the user wants flashcards specifically for learning a language —
  vocabulary, grammar, pronunciation, collocations, false friends, dialogues,
  idioms, or cultural notes in any target language. Trigger for phrases like
  "flashcards for [language]", "vocabulary cards", "make cards from this text",
  "help me memorize these words", "sentence mining", "grammar cards", or whenever
  the user shares a resource (word list, article, lesson, textbook page, image,
  URL) and wants to learn or memorize it in a foreign language. Prefer this skill
  over the general flashcard skill whenever a target language is mentioned or
  implied.
---
```

Replace it with:

```yaml
---
name: lang-flashcard
description: >
  Generate high-quality language learning flashcards from any input, following
  the SuperMemo Twenty Rules and language acquisition research. Use this skill
  whenever the user wants flashcards specifically for learning a language —
  vocabulary, grammar, pronunciation, collocations, false friends, dialogues,
  idioms, or cultural notes in any target language. Trigger for phrases like
  "flashcards for [language]", "vocabulary cards", "make cards from this text",
  "help me memorize these words", "sentence mining", "grammar cards", or whenever
  the user shares a resource (word list, article, lesson, textbook page, image,
  URL) and wants to learn or memorize it in a foreign language. Trigger whenever
  a target language is named or implied (e.g. Spanish, Japanese, German,
  "my French class").
---
```

This removes the "Prefer this skill over the general flashcard skill"
clause (no longer needed — `flashcard-router` handles disambiguation now)
and replaces it with a sharper statement of this skill's own trigger
vocabulary, per the spec's "Changes to `lang-flashcard/SKILL.md`" section.

- [ ] **Step 2: Replace the "Output format" section**

The current "Output format" section (the existing file's lines 309-347) is:

```markdown
## Output format

**Output a CSV artifact — no plain numbered lists, no HTML, no React components.** Cards are study material the user will copy into their flashcard app (Anki, Quizlet, etc.).

Output the cards as a `.csv` file artifact with the following header row and columns:

```
id,type,topic,front,back,notes
```

Column definitions:
- **id** — sequential integer starting at 1
- **type** — one of: `vocab`, `production`, `grammar`, `collocation`, `false-friend`, `dialogue`, `culture`, `pronunciation`, `mnemonic`
- **topic** — language and subject area (e.g. `Spanish / travel`, `Japanese / apologies`)
- **front** — the question or prompt side of the card; use `[...]` for cloze gaps
- **back** — the answer side of the card
- **notes** — optional: register, word class, IPA, pattern label, cultural tag — empty string if none
**CSV formatting rules:**
- Comma-separated, UTF-8 encoded
- Wrap any field containing commas, newlines, or double-quotes in double-quotes
- Escape internal double-quotes by doubling them (`""`)
- Multi-line answer content (e.g. dialogue alternatives) uses a **real newline** inside the quoted field — do NOT use the literal escape sequence `\n`
- Do not add extra blank lines between rows
**Example:**

```csv
id,type,topic,front,back,notes
1,vocab,Spanish / travel,"El vuelo sale a las [...] de la mañana.","ocho","time expressions use 'las' + number"
2,production,Spanish / travel,"""eight o'clock"" in Spanish (for time expressions)","las ocho",""
3,false-friend,Spanish,"""embarazada"" — false friend for English speakers?","Yes — means 'pregnant' NOT 'embarrassed'. 'embarrassed' = avergonzado/a",""
4,dialogue,Japanese / apologies,"Arriving late to a formal meeting in Japan — what do you say?","申し訳ございません。(Mōshiwake gozaimasen.) — deeply formal apology
Less formal: すみません、遅れました。(Sumimasen, okuremashita.)",""
```

After the artifact, add a short **summary** in chat (plain text):
- Total cards, breakdown by type
- Any items skipped (and why — too listy, out of scope, etc.)
- Any ⚠️ flags for unclear source material
- Reminder to shuffle / interleave cards during review
```

Replace it with:

```markdown
## Output format

See `../_shared/csv-output-format.md` for the full CSV schema, formatting
rules, and summary footer format — every flashcard skill in this repo
follows the same output contract.

This skill's `type` column values:
`vocab`, `production`, `grammar`, `collocation`, `false-friend`,
`dialogue`, `culture`, `pronunciation`, `mnemonic`.

**Example:**

```csv
id,type,topic,front,back,notes
1,vocab,Spanish / travel,"El vuelo sale a las [...] de la mañana.","ocho","time expressions use 'las' + number"
2,production,Spanish / travel,"""eight o'clock"" in Spanish (for time expressions)","las ocho",""
3,false-friend,Spanish,"""embarazada"" — false friend for English speakers?","Yes — means 'pregnant' NOT 'embarrassed'. 'embarrassed' = avergonzado/a",""
4,dialogue,Japanese / apologies,"Arriving late to a formal meeting in Japan — what do you say?","申し訳ございました。(Mōshiwake gozaimasen.) — deeply formal apology
Less formal: すみません、遅れました。(Sumimasen, okuremashita.)",""
```
```

Note: preserve the exact example CSV content from the original file
(four rows, same text) — only the surrounding prose changes. Double-check
the Japanese text reads `申し訳ございません` (gozaimasen, not gozaimashita)
to match the original file exactly when making this edit.

- [ ] **Step 3: Update the Step 4 quality checklist**

The current checklist (existing file's lines 294-307) is:

```markdown
## Step 4: Apply quality rules (SuperMemo Twenty Rules)

Before outputting, check each card:

- [ ] **One fact only** — if a question has two parts, split it.
- [ ] **Answer as short as possible** — trim every unnecessary word.
- [ ] **Noun-based prompts** where possible — "tarde — meaning" not "What does tarde mean?"
- [ ] **Difficulty matched to level** — desirable difficulty for intermediate/advanced; scaffolding and a high success rate for beginners (see Step 2.5). A card should be one the learner can *almost* always get with effort.
- [ ] **No bare word → translation** unless the user explicitly asks.
- [ ] **Context card present** for every new vocabulary item.
- [ ] **Production card paired** with recognition for active vocabulary — skip the pair for recognition-only items or to control deck size at beginner level (see Step 3B).
- [ ] **False friends flagged** — scan the target-language material for words that resemble native-language words with different meanings; generate a card for each one found (requires native language to be known).
- [ ] **Disambiguation cue added** when two cards could be confused.
```

Replace it with:

```markdown
## Step 4: Apply quality rules (SuperMemo Twenty Rules)

Before outputting, check each card against the shared baseline in
`../_shared/quality-rules.md`, plus these language-specific rules:

- [ ] **Difficulty matched to level** — desirable difficulty for
  intermediate/advanced; scaffolding and a high success rate for beginners
  (see Step 2.5).
- [ ] **Context card present** for every new vocabulary item.
- [ ] **Production card paired** with recognition for active vocabulary —
  skip the pair for recognition-only items or to control deck size at
  beginner level (see Step 3B).
- [ ] **False friends flagged** — scan the target-language material for
  words that resemble native-language words with different meanings;
  generate a card for each one found (requires native language to be
  known).
```

This removes the four bullets now covered by `_shared/quality-rules.md`
(one fact only, answer length, noun-based prompts, no bare word→translation
— this last one is the domain instance of the shared "no bare
term→definition pairs" rule) and the "Disambiguation cue" bullet (also now
in the shared file). "Difficulty matched to level" is kept here rather than
removed, because — per the spec — its level-specific elaboration in Step
2.5 is language-specific even though the shared file has a generic version
of the same bullet; restating it here with the pointer to Step 2.5 keeps
that connection visible.

- [ ] **Step 4: Verify the full file reads end-to-end without gaps**

Manual read-through check — read the entire updated
`skills/lang-flashcard/SKILL.md` top to bottom as if you are Claude Code
loading it fresh, with no other context:
- The frontmatter description no longer mentions "general flashcard skill".
- Step 0 through Step 3 (intake, reading the resource, deck sizing, level
  adaptation, card types) are byte-for-byte unchanged from before this task.
- The Output format section links to `../_shared/csv-output-format.md`
  and still lists the 9 language-specific `type` values inline.
- The Step 4 checklist references `../_shared/quality-rules.md` and still
  contains all 4 language-specific bullets (difficulty matched to level,
  context card present, production card paired, false friends flagged).
- The Edge cases section (end of file) is unchanged.
- Nothing in the file references a removed bullet (e.g. no leftover
  "see above" pointing at deleted content).

Run this check to confirm no leftover references to removed content:

```bash
grep -n "general flashcard skill" skills/lang-flashcard/SKILL.md
```

Expected: no output (the phrase should no longer appear anywhere in the
file).

```bash
grep -n "_shared" skills/lang-flashcard/SKILL.md
```

Expected: two matches — one for `../_shared/csv-output-format.md`, one for
`../_shared/quality-rules.md`.

- [ ] **Step 5: Commit**

```bash
git add skills/lang-flashcard/SKILL.md
git commit -m "Point lang-flashcard at shared CSV format and quality rules"
```

---

### Task 5: Update `CLAUDE.md` to document the new structure

**Files:**
- Modify: `CLAUDE.md`

**Interfaces:**
- Consumes: the existence of `skills/_shared/csv-output-format.md`,
  `skills/_shared/quality-rules.md`, and `skills/flashcard-router/SKILL.md`
  from Tasks 1-3 (this task documents them, doesn't create them).
- Produces: the final, authoritative repo-level convention doc that a
  future contributor (human or agent) reads before adding a new domain
  skill. Nothing downstream depends on this file's exact text, but it must
  accurately describe the state produced by Tasks 1-4.

- [ ] **Step 1: Update the "Structure" section**

The current section is:

```markdown
## Structure

Each flashcard variant lives in its own skill directory:

```
skills/
  <skill-name>/
    SKILL.md       # required: frontmatter (name, description) + instructions
    references/     # optional: supporting docs loaded on demand
    assets/         # optional: templates, scripts, examples
```

`SKILL.md` is the entry point Claude Code loads. The `description` frontmatter
field is what triggers the skill — write it so it clearly states what kind of
flashcards it produces and what phrases should invoke it (so multiple
flashcard skills can coexist without colliding).
```

Replace it with:

```markdown
## Structure

Each flashcard variant lives in its own skill directory:

```
skills/
  flashcard-router/
    SKILL.md       # fallback router for domain-ambiguous requests
  _shared/
    csv-output-format.md   # shared CSV schema, formatting rules, summary footer
    quality-rules.md       # shared domain-agnostic quality checklist
  <skill-name>/
    SKILL.md       # required: frontmatter (name, description) + instructions
    references/     # optional: supporting docs loaded on demand
    assets/         # optional: templates, scripts, examples
```

`SKILL.md` is the entry point Claude Code loads. The `description` frontmatter
field is what triggers the skill — write it so it clearly states what kind of
flashcards it produces and what phrases should invoke it.

`_shared/` holds reference material, not a skill — it has no `SKILL.md` of
its own, so Claude Code's skill loader never treats it as invokable.
Domain skills should link to `_shared/csv-output-format.md` and
`_shared/quality-rules.md` instead of restating their contents.

`flashcard-router/` is the fallback for requests where no domain is stated
or inferable. It discovers available domain skills by reading the `skills/`
directory at invocation time, so domain skills never need to know about
each other or maintain pairwise precedence rules.
```

- [ ] **Step 2: Update the "Current skills" section**

The current section is:

```markdown
## Current skills

- **lang-flashcard** (`skills/lang-flashcard/SKILL.md`) — language-learning
  flashcards (vocabulary, grammar, pronunciation, collocations, false
  friends, dialogues, culture notes) from any input material.
```

Replace it with:

```markdown
## Current skills

**Infrastructure (not a domain skill):**
- **flashcard-router** (`skills/flashcard-router/SKILL.md`) — fallback for
  flashcard requests with no stated or inferable domain; routes to the
  right domain skill or asks the user to disambiguate.

**Domain skills:**
- **lang-flashcard** (`skills/lang-flashcard/SKILL.md`) — language-learning
  flashcards (vocabulary, grammar, pronunciation, collocations, false
  friends, dialogues, culture notes) from any input material.
```

- [ ] **Step 3: Update the "Adding a new flashcard skill" section**

The current section is:

```markdown
## Adding a new flashcard skill

1. Create `skills/<new-name>/SKILL.md`.
2. Give it frontmatter with a `name` and a `description` that distinguishes
   it from existing flashcard skills (e.g. by domain: language vs. medical
   vs. coding vs. exam prep).
3. Keep the same general shape as `lang-flashcard`: intake questions, how to
   read the source resource, how to size the deck, card-type definitions,
   quality-control checklist, and the CSV output format — adapt content to
   the new domain.
4. If two skills could both plausibly match a request, make the
   `description` fields mutually exclusive (state which one takes priority
   for which trigger phrases), the way `lang-flashcard` says to prefer
   itself over the general flashcard skill when a target language is
   mentioned.
```

Replace it with:

```markdown
## Adding a new flashcard skill

1. Create `skills/<new-name>/SKILL.md`.
2. Give it frontmatter with a `name` and a `description` that states its
   own domain and trigger vocabulary clearly (e.g. by domain: language vs.
   medical vs. coding vs. exam prep). You do not need to reference other
   skills or rank against them — `flashcard-router` handles requests that
   don't clearly match any domain skill's description.
3. Keep the same general shape as `lang-flashcard`: intake questions, how to
   read the source resource, how to size the deck, card-type definitions,
   and a quality-control checklist — adapt content to the new domain.
4. For output format, link to `skills/_shared/csv-output-format.md` instead
   of restating the CSV schema and formatting rules; document only your
   skill's own `type` column values and a domain-specific worked example
   inline.
5. For quality rules, link to `skills/_shared/quality-rules.md` for the
   generic checklist and list only your domain-specific additions inline
   (the way `lang-flashcard`'s Step 4 does).
6. No router updates needed — `flashcard-router` discovers domain skills
   dynamically by reading the `skills/` directory, so a new skill is picked
   up automatically.
```

- [ ] **Step 4: Update the "Output convention" section**

The current section is:

```markdown
## Output convention

All flashcard skills in this repo output a CSV artifact with columns
`id,type,topic,front,back,notes` — keep new skills consistent with this so
downstream tooling (Anki/Quizlet import, etc.) doesn't need per-skill
handling.
```

Replace it with:

```markdown
## Output convention

All flashcard skills in this repo output a CSV artifact following the
shared schema and formatting rules in `skills/_shared/csv-output-format.md`
(columns `id,type,topic,front,back,notes`) — keep new skills consistent
with this so downstream tooling (Anki/Quizlet import, etc.) doesn't need
per-skill handling.
```

- [ ] **Step 5: Verify the full file reads end-to-end without gaps**

Manual read-through check — read the entire updated `CLAUDE.md` top to
bottom:
- The Structure section's directory tree includes `flashcard-router/` and
  `_shared/` alongside `<skill-name>/`.
- The Current skills section separates infrastructure (`flashcard-router`)
  from domain skills (`lang-flashcard`).
- The "Adding a new flashcard skill" steps no longer mention pairwise
  precedence/collision wording, and instead reference both shared files
  and state that no router update is needed.
- The Output convention section points at
  `skills/_shared/csv-output-format.md` rather than inlining the schema
  description (the column list may still appear as a quick-reference, but
  the shared file is cited as the source of truth).
- Cross-check this file's claims against what Tasks 1-4 actually produced:
  every path mentioned (`skills/flashcard-router/SKILL.md`,
  `skills/_shared/csv-output-format.md`, `skills/_shared/quality-rules.md`)
  must exist on disk.

Run this check to confirm every path `CLAUDE.md` references actually
exists:

```bash
test -f skills/flashcard-router/SKILL.md && \
test -f skills/_shared/csv-output-format.md && \
test -f skills/_shared/quality-rules.md && \
test -f skills/lang-flashcard/SKILL.md && \
echo "all referenced paths exist"
```

Expected output:
```
all referenced paths exist
```

- [ ] **Step 6: Commit**

```bash
git add CLAUDE.md
git commit -m "Document router and shared references in repo conventions"
```

---

### Task 6: Final cross-file consistency pass

**Files:**
- Read (no modification expected, but fix in place if a gap is found):
  `CLAUDE.md`, `skills/flashcard-router/SKILL.md`,
  `skills/_shared/csv-output-format.md`, `skills/_shared/quality-rules.md`,
  `skills/lang-flashcard/SKILL.md`

**Interfaces:**
- Consumes: all files produced by Tasks 1-5.
- Produces: confirmation (or in-place fixes) that the whole set is mutually
  consistent — this is the plan's equivalent of an integration test, since
  there is no automated suite to catch cross-file drift.

- [ ] **Step 1: Re-read the spec's "Testing / validation" section and check each item**

Re-open `docs/superpowers/specs/2026-06-24-flashcard-repo-scaling-design.md`
and confirm each validation bullet from its "Testing / validation" section:

- `flashcard-router/SKILL.md`'s dynamic discovery instructions are
  unambiguous about where to look (`skills/`, excluding `_shared/` and
  itself) — confirmed in Task 3, re-check here after all other edits landed
  in case any later task introduced confusion.
- `lang-flashcard/SKILL.md` still fully specifies card generation without
  the reader needing to guess at anything that moved to `_shared/` — open
  the file fresh and confirm there's no orphaned reference.
- `CLAUDE.md`'s "Adding a new flashcard skill" steps, followed literally,
  would produce a skill consistent with this design — walk through the 6
  steps from Task 5 mentally as if adding a hypothetical `medical-flashcard`
  skill, and confirm nothing is missing (frontmatter guidance, shape
  guidance, both shared-file links, no router-update requirement).

- [ ] **Step 2: Check for naming drift across files**

Run a search for the two shared file names and the router name across the
whole repo to make sure every reference uses the same exact filename and
path style (relative `../_shared/...` from skill directories, repo-rooted
`skills/_shared/...` from `CLAUDE.md`):

```bash
grep -rn "csv-output-format" --include="*.md" .
grep -rn "quality-rules" --include="*.md" .
grep -rn "flashcard-router" --include="*.md" .
```

Expected: every result uses the exact filename `csv-output-format.md` (no
`csv_output_format.md` or `output-format.md` variants), exact filename
`quality-rules.md`, and exact skill name `flashcard-router` (no
`flashcard_router` or `router` shorthand) — confirm no typos crept in
across the 5 files touched by Tasks 1-5. If any mismatch is found, fix it
in place and re-run the grep.

- [ ] **Step 3: Confirm git history is clean**

```bash
git log --oneline -8
git status
```

Expected: 5 new commits from Tasks 1-5 (shared CSV format, shared quality
rules, router skill, lang-flashcard update, CLAUDE.md update), plus the
earlier spec commit, with a clean working tree (no uncommitted changes) —
unless Step 2 above required an in-place fix, in which case commit that fix
now:

```bash
git add -A
git commit -m "Fix cross-file naming drift in flashcard skill references"
```

(Only run this commit if Step 2 actually found and fixed a mismatch —
otherwise skip it, since there is nothing to commit.)
