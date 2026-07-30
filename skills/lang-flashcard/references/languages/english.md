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
