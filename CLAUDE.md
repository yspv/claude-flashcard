# claude-flashcard

A collection of Claude Code skills for generating flashcards from a given
resource (text, URL, PDF, image, etc.).

## Structure

Each flashcard variant lives in its own skill directory, alongside the
router and shared references:

```
skills/
  flashcard-router/
    SKILL.md       # fallback router for domain-ambiguous requests
  _shared/
    csv-output-format.md   # shared CSV schema, formatting rules, summary footer
    quality-rules.md       # shared domain-agnostic quality checklist
  lang-flashcard/
    SKILL.md
    references/
      languages/   # per-language guidance modules (english.md, ...)
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

## Current skills

**Infrastructure (not a domain skill):**
- **flashcard-router** (`skills/flashcard-router/SKILL.md`) — fallback for
  flashcard requests with no stated or inferable domain; routes to the
  right domain skill or asks the user to disambiguate.

**Domain skills:**
- **lang-flashcard** (`skills/lang-flashcard/SKILL.md`) — language-learning
  flashcards (vocabulary, grammar, pronunciation, collocations, false
  friends, dialogues, culture notes) from any input material.

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
7. Add your skill to the "Current skills" list above (under Domain
   skills), so this file stays an accurate index.

## Adding a new language module to lang-flashcard

Language-specific guidance for `lang-flashcard` lives in
`skills/lang-flashcard/references/languages/<language>.md`, named by
the lowercase English name of the target language (e.g. `english.md`).
The skill checks for a matching module at invocation time (Step 0.5)
and loads it only when the target language matches — so adding a new
language module requires no SKILL.md, router, or CLAUDE.md changes.

Keep the same shape as `english.md`: hunting instructions for what to
find in source material, slanting rules for the existing card types
(A–I), an L1 interference derivation section (instructions plus a few
worked examples, never exhaustive per-L1 tables), and a short
language-specific quality checklist. Language modules must not define
new CSV `type` values.

## Output convention

All flashcard skills in this repo output a CSV artifact following the
shared schema and formatting rules in `skills/_shared/csv-output-format.md`
(columns `id,type,topic,front,back,notes`) — keep new skills consistent
with this so downstream tooling (Anki/Quizlet import, etc.) doesn't need
per-skill handling.
