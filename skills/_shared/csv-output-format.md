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

The `type` values below (`concept`, `application`, `dialogue`) are
illustrative only — they are not a suggested default set. Each domain
skill defines its own `type` enum, as described above.

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
