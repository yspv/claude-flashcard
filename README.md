# claude-flashcard

A Claude Code plugin that turns any input — text, a URL, a PDF, an image —
into a study deck of flashcards, exported as CSV ready for Anki, Quizlet, or
any spaced-repetition tool.

## Install

Add this repo as a plugin marketplace and install the plugin:

```
/plugin marketplace add yspv/claude-flashcard
/plugin install claude-flashcard
```

(Or point at a local checkout path instead of the GitHub slug if you're
developing against a clone.)

## Usage

Just ask, in any Claude Code session:

```
Make flashcards for my Spanish class from this article: <paste text>
```

```
Generate vocabulary cards from this word list: casa, perro, ventana, ...
```

Claude infers the right skill from your request. If the domain is stated or
obvious (a language, a named exam, etc.) the matching domain skill triggers
directly. If it's ambiguous — e.g. "make flashcards from this" with no
identifiable subject — the **flashcard-router** skill asks what domain
you're studying and hands off accordingly.

Every skill asks a few intake questions up front (target language and
level, native language, source material) before generating cards, and ends
by printing a CSV artifact plus a short summary of what was made.

## Structure

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

`SKILL.md` is the entry point Claude Code loads; its `description`
frontmatter is what triggers the skill, so it states the domain and the
phrases that should invoke it.

`_shared/` holds reference material, not a skill — it has no `SKILL.md`, so
it's never treated as invokable on its own. Domain skills link to
`_shared/csv-output-format.md` and `_shared/quality-rules.md` rather than
restating their contents.

`flashcard-router/` is the fallback for requests where no domain is stated
or inferable. It discovers available domain skills by reading the `skills/`
directory at invocation time, so domain skills never need to know about
each other.

## Current skills

**Infrastructure (not a domain skill):**
- **flashcard-router** — fallback for flashcard requests with no stated or
  inferable domain; routes to the right domain skill or asks the user to
  disambiguate.

**Domain skills:**
- **lang-flashcard** — language-learning flashcards (vocabulary, grammar,
  pronunciation, collocations, false friends, dialogues, culture notes)
  from any input material, following the SuperMemo Twenty Rules.

## Output format

All flashcard skills output a CSV artifact with columns
`id,type,topic,front,back,notes`, followed by a chat summary (card count by
type, anything skipped, a reminder to shuffle/interleave during review). See
`skills/_shared/csv-output-format.md` for the full schema and formatting
rules.

## Adding a new flashcard skill

1. Create `skills/<new-name>/SKILL.md`.
2. Give it frontmatter with a `name` and a `description` that states its
   own domain and trigger vocabulary clearly (e.g. by domain: language vs.
   medical vs. coding vs. exam prep). No need to reference or rank against
   other skills — `flashcard-router` handles requests that don't clearly
   match any domain skill's description.
3. Keep the same general shape as `lang-flashcard`: intake questions, how to
   read the source resource, how to size the deck, card-type definitions,
   and a quality-control checklist — adapt content to the new domain.
4. Link to `skills/_shared/csv-output-format.md` instead of restating the
   CSV schema; document only your skill's own `type` column values and a
   domain-specific worked example inline.
5. Link to `skills/_shared/quality-rules.md` for the generic checklist and
   list only your domain-specific additions inline (the way
   `lang-flashcard`'s Step 4 does).
6. No router updates needed — `flashcard-router` discovers domain skills
   dynamically by reading the `skills/` directory.
7. Add your skill to the "Current skills" list in `CLAUDE.md` (and this
   README) so the index stays accurate.

See `CLAUDE.md` for the full repo conventions.
