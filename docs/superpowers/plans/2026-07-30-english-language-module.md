# English Language Module Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an on-demand English guidance module to `lang-flashcard` that slants the existing card types at English-specific pain points, plus the loader step and repo documentation.

**Architecture:** A reference file at `skills/lang-flashcard/references/languages/english.md` is read by `lang-flashcard` via a new "Step 0.5" (after intake, before reading the resource) whenever the target language is English. No new skill, no router changes, no new CSV `type` values — the module only adds hunting instructions and slanting rules for existing card types A–I.

**Tech Stack:** Markdown only (Claude Code skill files). No build, no test framework — verification is via `test`/`grep` structural checks.

**Spec:** `docs/superpowers/specs/2026-07-30-english-language-module-design.md`

## Global Constraints

- Module path pattern is exactly `skills/lang-flashcard/references/languages/<language>.md`, named by the **lowercase English name** of the target language (first file: `english.md`).
- No new CSV `type` values; the shared output contract in `skills/_shared/csv-output-format.md` is untouched.
- No changes to `skills/flashcard-router/` or `skills/_shared/`.
- No hardcoded per-L1 interference tables — L1 handling is derivation instructions plus 2–3 worked examples only.
- The Step 0.5 loader in SKILL.md must be short (a few lines) and must not restate module content.

---

### Task 1: Create the English guidance module

**Files:**
- Create: `skills/lang-flashcard/references/languages/english.md`

**Interfaces:**
- Consumes: card type letters A–I and step numbers (Steps 0–4) as defined in `skills/lang-flashcard/SKILL.md`; the disambiguation-cue rule from `skills/_shared/quality-rules.md`.
- Produces: the module file at the exact path `skills/lang-flashcard/references/languages/english.md`, with a `## 6. English-specific quality checklist` section that Task 2's loader implicitly folds into Step 4. Tasks 2 and 3 refer to this path — do not rename it.

- [ ] **Step 1: Create the directory and write the module file**

Write `skills/lang-flashcard/references/languages/english.md` with exactly this content:

````markdown
# English — language-specific guidance module

Loaded by `lang-flashcard` (Step 0.5) when the target language is
English. Apply this guidance throughout Steps 1–4: while reading the
source material, hunt for every category below, and when the material
contains an instance, card it using the indicated card type. This
module only slants the existing card types (A–I) — it adds no new card
types and no new CSV `type` values.

## 1. Verb patterns: -ing vs to-infinitive

English catenative verbs dictate the form of the verb that follows,
and dictionary definitions rarely make this stick. Three groups:

- **verb + to-infinitive**: want, decide, hope, refuse, promise,
  agree, plan, offer, learn
- **verb + -ing**: enjoy, avoid, finish, suggest, mind, practice,
  consider, keep, give up
- **verb + either, with a meaning change**: stop, remember, forget,
  try, regret, go on, need

**Card guidance:** type C (grammar cloze) with the verb form as the
gap. For the meaning-change group, **always generate a contrast
pair** — two cards whose sentences force the two different meanings —
and add a disambiguation cue to each so the pair can't be confused
(per the shared quality rules).

```
Q: He stopped [...] (smoke) — he quit the habit for good.
A: smoking  (stop + -ing = quit an activity)
[Contrast card exists: "stop to smoke" = pause in order to smoke]
```

```
Q: He stopped [...] (smoke) — he paused his walk for a cigarette.
A: to smoke  (stop + to-infinitive = pause in order to do something)
[Contrast card exists: "stop smoking" = quit the habit]
```

## 2. Near-synonym discrimination

English is full of pairs whose meanings overlap but whose *situations
of use* don't. Known confusable sets — card the distinction whenever
even one member appears in the material:

say/tell · do/make · borrow/lend · bring/take · hear/listen ·
look/watch/see · speak/talk · teach/learn · travel/trip/journey ·
fun/funny · lose/miss · remember/remind · rob/steal · win/beat

**Card guidance:** never define these words in isolation. Use a cloze
(type A or C) where **only one member of the set fits**, and put the
deciding rule on the back.

```
Q: She [...] me that the meeting was cancelled. (say / tell?)
A: told  (tell + person; say + words: "She said that…")
```

```
Q: I need to [...] my homework before dinner. (do / make?)
A: do  (do = tasks & routine work; make = create/produce:
   make dinner, make a plan)
```

## 3. Collocation restrictions

English collocations are arbitrary and unforgiving — the
natural-sounding guess is often wrong. Hunt for:

- **verb + noun traps**: make a mistake (not *do*), do homework (not
  *make*), take a photo (not *make*), give a lecture, commit a crime
- **adjective + noun**: heavy rain (not *strong*), strong coffee (not
  *powerful*), fast food (not *quick*), high temperature (not *tall*)
- **fixed prepositions**: depend **on**, arrive **at/in**, married
  **to**, good **at**, interested **in**, responsible **for**

**Card guidance:** type D cloze on the restricted slot. On the back,
give the correct collocate **and name the likely wrong guess, marked
wrong** — the learner needs to unlearn the plausible error, not just
learn the right form.

```
Q: I [...] a mistake in the report.  (verb)
A: made  (make a mistake — NOT "do a mistake")
```

## 4. Classic English pain points

### Phrasal verbs

Treat each phrasal verb as a vocabulary item in its own right — its
meaning is rarely the sum of its parts. Prefer phrasal verbs actually
present in the source material over generic lists.

- Type A cloze in context for each one; add a type B production card
  for high-frequency ones the learner should actively use.
- Note **separable vs inseparable** on the back when it matters:

```
Q: I don't know this word — I'll look it [...] in the dictionary.
A: up  (look up = search for information; separable: "look it up")
```

### Articles (a / an / the / ∅)

Type C cloze with the article as the gap. The back states the
specific rule instance at play, never the whole article system:

```
Q: I saw [...] dog in the park. [...] dog was chasing a ball.
A: a … The  (first mention → a; already-mentioned → the)
```

Common ∅ cases worth carding: generalizations ("∅ dogs are loyal"),
meals ("have ∅ breakfast"), languages, most proper names.

### Spelling–pronunciation mismatch & word stress

Type H cards. Per the existing type H convention: plain-language
respelling first, IPA in parentheses.

- **Silent letters**: comb, receipt, island, Wednesday, salmon
- **Heteronyms**: read (present) vs read (past); record (noun) vs
  record (verb)
- **Stress-shift pairs**: PREsent (n.) / preSENT (v.), REcord (n.) /
  reCORD (v.)

```
Q: "receipt" — pronunciation
A: rih-SEET (/rɪˈsiːt/) — the "p" is silent
```

### Irregular verbs

Card only irregular forms encountered in or implied by the material —
never bare paradigm tables (a paradigm is several facts; split it).
Type C cloze in a sentence:

```
Q: Yesterday she [...] (teach) us the past tense.
A: taught  (teach – taught – taught)
```

## 5. L1 interference derivation

When the learner's native language is known (from Step 0 intake), run
this derivation pass and weight card generation toward the areas it
flags. These are instructions for deriving interference from *any*
L1 — not a lookup table.

Ask, for the learner's L1:

1. **Does the L1 have articles?** If not → increase the share of
   article cards (§4).
2. **Which English phonemes are missing from the L1** (/θ/, /ð/,
   /w/–/v/, /ɪ/–/iː/, /æ/–/e/…)? → minimal-pair type H cards for each
   missing contrast.
3. **How does the L1 mark tense/aspect?** → target the distinctions
   English makes that the L1 merges (most often past simple vs
   present perfect).
4. **Known false friends between the L1 and English** → type E cards
   (the existing Step 3E rule, run as part of this pass).
5. **What word-order or preposition patterns would the L1 transfer
   wrongly?** → grammar/collocation cards for those specific calques.

**Worked examples of the derivation** (showing the method — not an
exhaustive table):

- **L1 = Russian**: no articles → heavy article-card share; no
  /θ/–/s/ contrast → think/sink, mouth/mouse minimal pairs; /w/–/v/
  confusion → west/vest; one past tense → past simple vs present
  perfect contrast cards; false friends → "magazine" (магазин =
  shop), "fabric" (фабрика = factory).
- **L1 = Spanish**: has articles but uses them differently for
  generics ("la vida" vs "∅ life") → ∅-article cards; no /ɪ/–/iː/
  contrast → ship/sheep minimal pairs; false friends → "embarrassed"
  (embarazada = pregnant), "actually" (actualmente = currently);
  preposition calques → depend **on** (not "of", from "depender de").

## 6. English-specific quality checklist

Add these to the Step 4 language-specific checks when this module is
loaded:

- [ ] Meaning-change verb-pattern cards come in contrast pairs with
  disambiguation cues.
- [ ] Near-synonym cards test the *choice between* members, not
  definitions.
- [ ] Phrasal verbs are carded as units, with separability noted when
  it matters.
- [ ] Collocation cards name the likely wrong guess, marked wrong.
- [ ] L1 derivation pass performed when the native language is known.
````

- [ ] **Step 2: Verify structure**

Run:

```bash
test -f skills/lang-flashcard/references/languages/english.md && grep -c '^## ' skills/lang-flashcard/references/languages/english.md
```

Expected: prints `6` (six `##` sections).

Run:

```bash
grep -n 'type: ' skills/lang-flashcard/references/languages/english.md
```

Expected: no output (module defines no new CSV `type` values — Global Constraints).

- [ ] **Step 3: Commit**

```bash
git add skills/lang-flashcard/references/languages/english.md
git commit -m "Add English language guidance module for lang-flashcard"
```

---

### Task 2: Add the Step 0.5 loader to lang-flashcard SKILL.md

**Files:**
- Modify: `skills/lang-flashcard/SKILL.md` (insert between the end of Step 0 and the `## Step 1` heading; Step 0 currently ends with the line `If all of these are already clear from the conversation, skip asking and proceed.`)

**Interfaces:**
- Consumes: the module path `references/languages/english.md` created in Task 1 (referenced generically as `references/languages/<language>.md`).
- Produces: a `## Step 0.5: Load language-specific guidance` section that future language modules rely on — the path pattern and "lowercase English name" naming rule stated here are the contract for all future `<language>.md` files.

- [ ] **Step 1: Insert the Step 0.5 section**

In `skills/lang-flashcard/SKILL.md`, immediately after Step 0's closing line (`If all of these are already clear from the conversation, skip asking and proceed.`) and the `---` separator that follows it, insert this new section (followed by its own `---` separator, matching the file's existing section style):

```markdown
## Step 0.5: Load language-specific guidance

As soon as the target language is known, check whether
`references/languages/<language>.md` exists, where `<language>` is the
lowercase English name of the target language (e.g. `english.md`). If
it exists, read it fully **before Step 1** — it contains hunting
instructions that shape what to extract from the material — and apply
its guidance throughout Steps 1–4, including folding its quality
checklist into Step 4. If no file exists for the language, skip this
step and proceed as normal.
```

- [ ] **Step 2: Verify placement and content**

Run:

```bash
grep -n '^## Step' skills/lang-flashcard/SKILL.md | head -5
```

Expected: `Step 0`, `Step 0.5`, `Step 1` appear in that order.

Run:

```bash
grep -n 'references/languages/<language>.md' skills/lang-flashcard/SKILL.md
```

Expected: exactly one match, inside the Step 0.5 section.

- [ ] **Step 3: Commit**

```bash
git add skills/lang-flashcard/SKILL.md
git commit -m "Load per-language guidance modules in lang-flashcard (Step 0.5)"
```

---

### Task 3: Document the language-module pattern in CLAUDE.md

**Files:**
- Modify: `CLAUDE.md` (two spots: the Structure section's directory tree + a new subsection after "Adding a new flashcard skill")

**Interfaces:**
- Consumes: the path pattern `skills/lang-flashcard/references/languages/<language>.md` and naming rule from Tasks 1–2.
- Produces: repo documentation only; nothing depends on it programmatically.

- [ ] **Step 1: Update the Structure section tree**

In `CLAUDE.md`, in the Structure section's code block, the generic skill entry currently reads:

```
  <skill-name>/
    SKILL.md       # required: frontmatter (name, description) + instructions
    references/     # optional: supporting docs loaded on demand
    assets/         # optional: templates, scripts, examples
```

Immediately **above** those `<skill-name>/` lines, add a concrete entry for lang-flashcard's language modules:

```
  lang-flashcard/
    SKILL.md
    references/
      languages/   # per-language guidance modules (english.md, ...)
```

- [ ] **Step 2: Add the "Adding a new language module" subsection**

In `CLAUDE.md`, immediately after the numbered list that ends the "Adding a new flashcard skill" section (its last item is item 7, about updating the "Current skills" list) and before the "## Output convention" heading, insert:

```markdown
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
```

- [ ] **Step 3: Verify**

Run:

```bash
grep -c 'references/languages' CLAUDE.md && grep -c 'languages/' CLAUDE.md
```

Expected: `1` then `2` — the full path appears once (in the new subsection; the tree splits `references/` and `languages/` across lines), and `languages/` appears twice (tree + subsection).

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "Document per-language guidance modules in CLAUDE.md"
```

---

### Task 4: End-to-end structural check

**Files:**
- No file changes — verification only.

**Interfaces:**
- Consumes: all three previous tasks' outputs.
- Produces: confirmation that paths referenced across files agree.

- [ ] **Step 1: Cross-file consistency check**

Run:

```bash
test -f skills/lang-flashcard/references/languages/english.md \
  && grep -q 'references/languages/<language>.md' skills/lang-flashcard/SKILL.md \
  && grep -q 'references/languages/<language>.md' CLAUDE.md \
  && grep -q 'Step 0.5' skills/lang-flashcard/SKILL.md \
  && echo OK
```

Expected: `OK`.

- [ ] **Step 2: Confirm untouched files**

Run:

```bash
git status --porcelain skills/flashcard-router skills/_shared
```

Expected: no output (router and shared files untouched — Global Constraints).
