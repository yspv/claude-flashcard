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

Before doing anything else, list the contents of the `skills/` directory —
that is, this skill's own parent directory (the directory containing the
`flashcard-router/` folder this `SKILL.md` lives in) — to see which domain
skills currently exist. Do this fresh on every invocation — never hardcode
a list of domains or skill names in this file. The whole point of this
router is that adding a new domain skill requires zero edits here.

Exclude from consideration:
- `_shared/` — this is a reference directory, not a skill (it has no
  `SKILL.md`).
- `flashcard-router/` — this skill itself.

For each remaining directory, read its `SKILL.md` frontmatter
`description` to understand what domain and trigger phrases it covers.

If sibling skills cannot be enumerated for any reason (e.g. this skill is
installed in a context that flattens or otherwise doesn't expose the
repo's directory layout), do not fail or error out — skip discovery
entirely and go straight to asking the user the disambiguation question in
Step 2.

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
