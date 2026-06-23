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
