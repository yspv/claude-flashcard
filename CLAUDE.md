# claude-flashcard

A collection of Claude Code skills for generating flashcards from a given
resource (text, URL, PDF, image, etc.).

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

## Current skills

- **lang-flashcard** (`skills/lang-flashcard/SKILL.md`) — language-learning
  flashcards (vocabulary, grammar, pronunciation, collocations, false
  friends, dialogues, culture notes) from any input material.

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

## Output convention

All flashcard skills in this repo output a CSV artifact with columns
`id,type,topic,front,back,notes` — keep new skills consistent with this so
downstream tooling (Anki/Quizlet import, etc.) doesn't need per-skill
handling.
