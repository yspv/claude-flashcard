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
 
# Language Flashcard Skill
 
Generate language learning flashcards from any resource. Applies SuperMemo's
Twenty Rules, cognitive science best practices (retrieval practice, desirable
difficulty, interleaving), and a rich set of card types designed specifically
for language acquisition.
 
---
 
## Step 0: Intake
 
At the start of every session, if the user hasn't already stated these, ask (all
at once, not one by one):
 
1. **Target language** — what are they learning?
2. **Native language** — what do they speak natively? (Needed for false friends,
   example sentences, and production cards.)
3. **Proficiency level** — roughly beginner, intermediate, or advanced (or a CEFR
   level like A2 / B1 / C1 if they know it). This drives card difficulty, how much
   scaffolding to give, and which direction to emphasize. If unstated, infer from
   the material and the user's phrasing, default to beginner, and say which level
   you assumed so they can correct it.
4. **Input** — what material are they working from? (Paste it, upload a file, or
   give a URL. Or just name a topic and cards will be generated from knowledge.)
If all of these are already clear from the conversation, skip asking and proceed.
 
---
 
## Step 0.5: Load language-specific guidance

As soon as the target language is known, check whether
`references/languages/<language>.md` exists, where `<language>` is the
lowercase English name of the target language (e.g. `english.md`). If
it exists, read it fully **before Step 1** — it contains hunting
instructions that shape what to extract from the material — and apply
its guidance throughout Steps 1–4, including folding its quality
checklist into Step 4. If no file exists for the language, skip this
step and proceed as normal.

---
 
## Step 1: Read and understand the resource
 
Fully read and comprehend the material before generating any cards.
 
- **URL** → fetch with web fetch tool
- **PDF** → read with pdf-reading skill
- **Word document** → read with docx skill
- **Image** → describe visually (textbook pages, notes, menus, signs, etc.)
- **Topic only** → generate cards from your knowledge of the language
Identify:
- Vocabulary items (content words first, then function words)
- Grammar patterns and structures present
- Collocations and fixed expressions
- Culturally loaded items (idioms, pragmatic markers, register shifts)
- Anything the user is most likely to encounter in real use
---
 
## Step 2: Decide how many cards to make
 
| Input size | Cards |
|---|---|
| A short word list (< 10 items) | 2–3 cards per item |
| A paragraph or short text | 8–15 cards |
| A medium article or lesson (500–1500 words) | 15–30 cards |
| A full chapter or long document | 25–50 cards |
 
Err toward more cards (split complex items) over fewer (cramming multiple facts
into one card).
 
---
 
## Step 2.5: Adapt to the learner's level
 
Proficiency changes *how* a card is built, not just which words are chosen. Apply
these adjustments throughout Step 3.
 
| Dimension | Beginner (A1–A2) | Intermediate (B1–B2) | Advanced (C1–C2) |
|---|---|---|---|
| **Cloze sentence** | Short, high-frequency, one new element; rest of the sentence already known | Authentic full sentences from the material | Long, idiomatic, register-marked sentences |
| **What's hidden** | One short, well-signposted gap; keep surrounding scaffolding | Target word or phrase | Larger gaps, subtler items (particles, aspect, connectors) |
| **Glossing** | Always gloss the full sentence in the native language | Gloss only the tricky part | Minimal or no gloss; stay in the target language |
| **Direction emphasis** | Recognition-heavy (see Step 3B on when to skip production) | Balanced recognition + production | Production-heavy; push active recall |
| **Card types to favor** | A (vocab cloze), B (production), H (pronunciation) | add C, D, E | add F, G, I and nuance/register cards |
 
The "not trivially guessable / desirable difficulty" rule in Step 4 is **calibrated
to level**: for beginners, scaffolding and a high success rate matter more than
difficulty — a cloze they can't fill just fails silently. For advanced learners,
lean into difficulty. Aim for a card the learner can *almost* always get, with effort.
 
---
 
## Step 3: Generate cards
 
Use the **card types** below. Mix types — a good set includes recognition,
production, grammar, and at least one or two deeper types (collocation, dialogue,
cultural note, false friend) where the material supports it.
 
---
 
### Card types
 
#### A. Vocabulary-in-context (cloze) — *default for new words*
 
Show the word in a real sentence with a gap. Teaches meaning, grammar, and usage
simultaneously. **Always prefer this over bare word → translation pairs.**
 
If the user provided source material, extract real sentences from it (sentence
mining). Real-context sentences produce stronger memory traces than invented ones.
 
```
Q: Ella siempre llega […] a las reuniones. (She always arrives ___ to meetings.)
A: tarde  (late — adverb, neutral register)
[Word class: adverb | Register: neutral]
```
 
#### B. Production card (reverse)
 
Recognition (seeing the foreign word and knowing its meaning) is easier than
production (retrieving the foreign word from your native language). Both are needed
for fluency, so pair recognition with production for any word the learner needs to
actively *use*.
 
```
Recognition: Q: tarde — meaning    A: late (adverb, neutral)
Production:  Q: "late" in Spanish (adverb, neutral)   A: tarde
```
 
**When to skip the production pair** (to keep the deck from doubling unnecessarily):
- The item is recognition-only — rare or literary words the learner only needs to
  *understand* when reading/listening, not produce.
- Beginner level with a large word list: pair production only for the highest-value,
  most frequent words first; the rest can stay recognition-only until later.
- The production direction is trivial or identical (e.g. an international cognate).
For advanced learners, bias the other way: production cards are where the real work is.
 
#### C. Grammar pattern card (cloze)
 
For conjugations, cases, articles, tenses, word order, or any grammatical rule.
Always use a real example sentence — never an abstract rule in isolation.
 
```
Q: Es importante que yo […] español todos los días.
A: hable  (subjunctive of hablar)
[Pattern: subjunctive after impersonal expressions with "que"]
```
 
```
Q: Sie […] gestern nicht […]. (She ___ not come yesterday.) [Perfekt of "kommen"]
A: ist … gekommen
[Pattern: sein + past participle for motion / state-change verbs]
```
 
#### D. Collocation card
 
Words that travel together naturally. Native speakers don't say "do an effort" —
they "make an effort." Collocations are invisible to dictionaries but essential
for sounding natural. Frame these in the **target language**, consistent with the
other card types: give the cloze or prompt in the target language and the natural
partner word(s) as the answer.
 
```
Q: ___ una decisión  (verb that goes with "decisión" — Spanish)
A: tomar  (tomar una decisión — "to make a decision", lit. "to take")
```
 
```
Q: "prendre" — 3 common collocations (French)
A: prendre une décision / prendre un café / prendre le bus
```
 
#### E. False friend card
 
Words in the **target language** that look or sound like a word in the native
language but mean something different. These are high-interference items and
need explicit cards.
 
Frame the card from the target language outward — the learner encounters a
target-language word, it triggers a false match with their native language, and
the card corrects that assumption.
 
```
Q: Spanish "embarazada" — what does it mean? (⚠ looks like "embarrassed")
A: "pregnant" — NOT "embarrassed"
   "embarrassed" in Spanish = avergonzado/a
```
 
```
Q: French "librairie" — what does it mean? (⚠ looks like "library")
A: "bookshop" — NOT "library"
   "library" in French = bibliothèque
```
 
Proactively scan the material for false friends even when the user hasn't asked
— if the target language contains a word that resembles a native-language word
with a different meaning, generate a false-friend card for it. Only generate
these cards when the learner's native language is known.
 
#### F. Dialogue / situational card
 
A short exchange or situational prompt. The answer is **a phrase the learner
produces** in a given situation — greetings, requests, apologies, refusals,
discourse markers. If the card tests *what to say*, it belongs here.
 
> **Boundary with type G:** Type F asks the learner to produce an utterance ("what
> do you say when…"). Type G tests an underlying norm or rule ("when is X
> appropriate / rude"). If the answer is a sentence to speak, use F; if the answer
> is a fact about usage, use G. Don't make both for the same item.
 
```
Q: How do you politely ask to repeat something in German?
A: "Könnten Sie das bitte wiederholen?" (formal)
   "Kannst du das nochmal sagen?" (informal)
[Situation: didn't catch what someone said]
```
 
```
Q: You arrive 20 minutes late to a meeting in Japan. What do you say?
A: "遅れてすみません。" (Okurete sumimasen.) — I'm sorry for being late.
[Note: より formal alternatives exist, e.g. 申し訳ございません in business contexts]
```
 
#### G. Cultural / pragmatic note card
 
For items where cultural knowledge changes meaning, appropriateness, or impact:
register shifts, taboo topics, honorifics, gestures, indirect speech. The answer
is **a rule or fact about usage** — *when* or *whether* something is appropriate —
not a phrase to recite (that's type F).
 
```
Q: In Japanese, when is it rude to say "no" directly?
A: In most social/professional contexts — indirect refusals are preferred
   (e.g., "chotto..." / "ちょっと…" — "It's a bit...") to avoid causing loss of face.
[Culture: Japanese indirect communication style]
```
 
```
Q: French "tu" vs "vous" — when to use each?
A: "vous" = formal/plural; "tu" = friends, family, children, peers
   Switching to "tu" uninvited can seem rude in professional contexts.
```
 
#### H. Pronunciation / phonetics card
 
For sounds that don't exist in the learner's native language, minimal pairs, or
words with tricky stress.
 
Most learners don't read IPA. Lead with a plain-language description or a
native-language respelling, and add IPA in parentheses as a supplement, not the
main cue — unless the learner has signaled they read IPA, in which case use it
freely. When in doubt, give both.
 
```
Q: French "u" sound — how to produce it
A: Round lips as if saying "oo", but try to say "ee" → /y/
   Examples: tu, rue, vu
```
 
```
Q: Minimal pair — Spanish "pero" vs "perro"
A: pero /ˈpe.ɾo/ (flap r) = but
   perro /ˈpe.ro/ (trill r) = dog
```
 
For tonal languages (Mandarin, Vietnamese, Thai, Cantonese), always include:
- Tone mark + tone number
- IPA or description of the pitch contour
- A short mnemonic for the tone shape
```
Q: Mandarin tone 3 (third tone) — shape and example
A: Low dipping tone: starts mid, dips low, rises slightly → /˨˩˦/
   Example: 我 wǒ (I/me)
   Mnemonic: like a sigh — goes down before coming back up
```
 
#### I. Mnemonic / memory hook card
 
For vocabulary that keeps slipping. A vivid image linking the sound of the word
to its meaning encodes far faster than repetition.
 
```
Q: German "Eichhörnchen" — mnemonic
A: Sounds like "I–corn–hen" → picture a hen on an ear of corn up in an oak tree.
   Meaning: squirrel
```
 
Generate these only for words flagged as difficult, or when the user requests them.
 
---
 
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
---
 
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
4,dialogue,Japanese / apologies,"Arriving late to a formal meeting in Japan — what do you say?","申し訳ございません。(Mōshiwake gozaimasen.) — deeply formal apology
Less formal: すみません、遅れました。(Sumimasen, okuremashita.)",""
```
---
 
## Edge cases
 
**Word list with no sentences** — generate context sentences yourself. Never leave
vocabulary as bare pairs.
 
**Source material in target language** — mine real sentences directly from it for
cloze cards. Always prefer real sentences over invented ones.
 
**Script-based languages (Arabic, Japanese, Chinese, Korean, etc.)** — include both
the script and romanization on the answer side until the learner specifies they
want script-only cards.
 
**Abstract vocabulary** — for abstract nouns (justice, freedom, nostalgia) that
can't be shown in a physical scene, anchor to a culturally resonant example
sentence or ask the user if they have a personal context in mind.
 
**Very dense material** — don't card every sentence. Prioritize: high-frequency
words, grammar patterns the learner hasn't encountered before, culturally loaded
items, and false friends.
