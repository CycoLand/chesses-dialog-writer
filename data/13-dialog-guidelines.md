# Doc 13 — Dialog Writing Guidelines
**For: Chesses Campaign Mode — Character Dialog**
**Applies to: All `VariantCharacter` dialog, authored via the Dialog Writer tool**

---

## Authoring workflow — the Dialog Writer tool

Dialog content is no longer written or edited directly in
`CharacterCatalog.kt`. That file is a **generated build artifact**. The
source of truth is one JSON file per character in
`chess-variants-app/dialog-data/<variantId>.json`, authored through the
**Dialog Writer** — a local, single-file HTML tool at
`chess-variants-app/tools/dialog-writer/dialog-writer.html`.

**To write or revise a character's dialog:**

1. Open `dialog-writer.html` in Chrome or Edge (it uses the File System
   Access API; other browsers can't open/save the folder directly, but
   `DOWNLOAD JSON` still works as a fallback export).
2. Click **OPEN chess-variants-app/** and select the `chess-variants-app`
   folder itself.
3. Pick the character from the sidebar. The sidebar's status dot is
   computed live from the same validation `VariantCharacter.init{}` runs at
   runtime — green means the character would pass, red means it wouldn't.
4. Each trigger is shown in three tiers (Required / Universal Optional /
   Variant-Specific), each with its knowledge-column description inline, a
   live line-count badge, and a non-blocking linter that flags guideline
   violations (em-dashes, 3+ sentence lines, sub-5-word sentences, ALL CAPS,
   exclamation overuse) as you type. The reference panel (▸ REFERENCE) shows
   the character's design doc and this guidelines doc side by side so you
   never have to alt-tab.
5. Click **SAVE CHARACTER** to write the JSON back to disk. (If the browser
   can't get write access, use **DOWNLOAD JSON** and move the file into
   `dialog-data/` manually.)

**To turn edited JSON into a working build**, run the generator:

```
node chess-variants-app/tools/dialog-writer/generate-catalog.mjs
```

This regenerates `CharacterCatalog.kt` and `dialog-status.md` from every file
in `dialog-data/`. It validates every character first and refuses to write
anything if any character is invalid (use `--force` only for a mid-session
checkpoint; never commit a `--force` build). `dialog-status.md` is fully
derived from this validation now — there is nothing to update by hand.

**One-time/occasional migration**, if `dialog-data/` ever needs to be
reseeded from the current `CharacterCatalog.kt` (e.g. after a manual Kotlin
edit slipped through):

```
node chess-variants-app/tools/dialog-writer/parse-catalog.mjs
```

This is read-only with respect to the Kotlin file — it never writes it.

---

Read this document before writing a single line of dialog. Read it again
after writing a batch and before committing. The self-review checklist at
the end is not optional.

Before starting on a character:

1. **Check the sidebar status dot** in the Dialog Writer (or
   [`dialog-status.md`](dialog-status.md), which mirrors it) to confirm the
   character is listed as Pending and not already in progress.

2. **Read the character's design document** in
   `chess-variants-app/docs/character-designs/`. The design doc defines who
   the character is — their personality, backstory, what they want, what they
   don't know, and their relationship to the player. Dialog written without
   reading it will be technically rule-compliant and still wrong: the lines
   will sound like a generic character in this variant's situation rather than
   this specific person. The design doc is not optional background. It is the
   primary source for the character's voice, preoccupations, vocabulary, and
   the specific details — a record they keep, a relative they disagree with,
   an image they've claimed — that make a line attributable to them and no
   one else.

This document covers four things:

1. **The trigger knowledge table** — what each trigger is and is not allowed
   to know. Violating this produces dialog that is factually wrong at runtime.
2. **The writing rules** — craft standards every line must meet.
3. **The AI writing tell reference** — specific phrases and patterns that
   signal AI-generated text. Scrub all of them before committing.
4. **Sourced character rules** — additional discipline for characters whose
   voice is drawn from a real person, fictional character, or specific
   cultural archetype.

**The tradition this document is working in:** The model for this kind of
character writing is Toby Fox's work in *Undertale* and *Deltarune*. Those
games are the clearest existing examples of what it looks like to make small
cast of characters feel irreducibly specific — not because of elaborate
backstory, but because of the precise, concrete, slightly embarrassing detail
that could only belong to that person. Papyrus has never tasted his own
spaghetti. Spamton painted a blue sky on the back wall of his dumpster shop.
These are the two things to internalize about why those characters are
unforgettable: the detail is specific, and it costs the character something
to have it be true. If you haven't played those games, do. They are the
reference point for everything this document is asking you to do.

---

## Part 1 — Trigger Knowledge Table

Every trigger fires in a specific context. A line may only reference facts
that are **guaranteed true** when that trigger fires. Referencing anything
outside the knowledge column produces false dialog — a character celebrating
a mine explosion that never happened, naming a piece type that was never
captured, referencing checks in a game that has no checks.

The **event-guarantee rule:** a line may claim a specific event occurred
only if that event is guaranteed by the trigger definition — not merely
possible. A `PLAYER_GOOD_MOVE` line must not say "nice capture" because
no capture is guaranteed to have occurred. A `REINFORCEMENT_DEPLOYED` line
must not say "three fresh knights" because the count and type are not known.

| Trigger | When it fires | What the character knows | What they do NOT know |
|---|---|---|---|
| `ENCOUNTER_START` | Once, after the warning screen | The variant is about to begin. Nothing about the board. | Any move, position, or piece arrangement. |
| `PLAYER_GOOD_MOVE` | Eval shifts >1.5 pawns in character's favour | The position just improved for the character. | What moved, where it moved, whether a capture occurred, which piece was involved. |
| `PLAYER_BAD_MOVE` | Eval shifts >1.5 pawns against the character | The position just worsened for the character. | Same as PLAYER_GOOD_MOVE — no specifics. |
| `OPPONENT_WINS` | Game ends, character won | The character won. | How the win was achieved — no checkmate, explosion, extinction, or specific move reference. |
| `PLAYER_WINS` | Game ends, character lost | The character lost. | How the loss was achieved. |
| `REINFORCEMENT_DEPLOYED` | Elite reinforcement pieces placed mid-fight | New pieces just arrived on the board. | How many, what type, which squares. |
| `ANTI_CHEESE_REINFORCEMENT` | Pre-game, player had a mate-in-1 from loadout | The player's starting formation was suspiciously aggressive. A blocking pawn is being placed. | Which piece, file, rank, or square was the threat. |
| Variant-specific triggers | Defined by `dialogEventDetector` in each variant file | Exactly what the detector passes — read the variant's KDoc. | Anything the detector does not pass. |

**When in doubt, leave it out.** If you are not certain a fact is
guaranteed by the trigger, do not reference it. Express the character's
reaction to the *situation* (improved position, worsening position,
arrival of backup) without describing the mechanism that produced it.

---

## Part 2 — The Writing Rules

### The foundational question

Before applying any rule, answer this: **what does this character want that
they'll never say directly?**

Every memorable character is built around a want, a wound, or an obsession
that organises their whole self-image and leaks into everything they say.
Not their role in the story — the specific thing they privately care about,
reach toward, or can't let go of.

**Build characters from obsessions, not traits.** A character who is
"enthusiastic" is a type. A character who is specifically obsessed with the
Royal Guard, with being popular, with his spaghetti, and with his puzzles
being completed properly — that is a person. The obsession generates lines
you couldn't predict and couldn't mistake for anyone else. If you can fully
describe a character without naming a single specific thing they own, fear,
return to, or have embarrassingly strong opinions about, you haven't written
a character yet.

The character's design document defines their obsessions. Before writing a
single line, be able to answer: what does this character specifically care
about that has nothing to do with chess? What would they still be thinking
about if they'd never played a game?

The rules below are about quality and discipline. This question is about
whether you're writing a character at all.

---

### Rule 1 — Each line is 1-2 sentences. Never 3.

Three short sentences in a row is one of the most reliable AI writing
tells in existence. It produces a staccato rhythm that reads as generated,
not spoken:

*"Good. Not what I needed. I'll note that."*

No real person speaks in three clipped declaratives in a row. If you
find yourself writing three sentences, either merge them into two, or cut
one entirely. The rule is absolute: maximum two sentences per line.

---

### Rule 2 — Short sentences are the single most common failure mode

This rule exists because short sentences are the failure mode that
appears most often in practice, across every character and every
trigger. Read it carefully before writing a single line.

**Every sentence in a line must be at least 5 words long.**

There are no exceptions to this floor. A sentence of fewer than 5 words
is an injective sentence — a fragment used for rhetorical punch:
"Good." "Fine." "It wasn't." "I play them anyway." "The planning held."
"Now you know." "I respect that." These feel punchy in isolation and
read as AI-generated at scale. They accumulate. A set of 10 lines with
four or five of them will read as generated regardless of how good the
other lines are.

**The most common violations — know these by sight:**

These all appear to be fine on a quick read. They are not.

| Violating line | Problem | Fix |
|---|---|---|
| `"I know the refutations. I play them anyway."` | "I play them anyway." = 4 words | `"I know every refutation for every gambit I play, and I play them anyway."` |
| `"The planning held."` as a second sentence | 3 words | Merge: `"…and the planning held."` |
| `"It wasn't."` as a second sentence | 2 words | Merge: `"…and it wasn't."` |
| `"Now you know."` as a second sentence | 3 words | Merge: `"…and now you know."` |
| `"I respect that."` as a second sentence | 3 words | Merge: `"…which I respect."` |
| `"That looks safe. It isn't."` | Both sentences under 5 words | `"It looks safe from where you're sitting, and it isn't."` |
| `"You sacrificed. Now I'm watching…"` | "You sacrificed." = 2 words | `"You've sacrificed, and now I'm watching…"` |
| `"Good. Show me the…"` | "Good." = 1 word | Drop it: `"Show me the…"` |
| `"Down a piece. The second move…"` | "Down a piece." = 3 words | `"Down a piece, and the second move…"` |
| `"More pieces. Each one is…"` | "More pieces." = 2 words | `"More pieces on the board, and each one is…"` |

**The fix is always the same:** merge the short sentence into the one
beside it using a comma or "and." If it cannot be merged, extend it
until it reaches 5 words or more. If it adds nothing when extended,
cut it entirely.

Short sentences are permitted **only** under these conditions:

- The sentence is at least 5 words long (it is then no longer a
  short injective sentence by this rule's definition)
- At most one sentence per line may be close to the 5-word floor —
  the other sentence must be substantially longer
- Never two consecutive short sentences anywhere in the same line —
  two short sentences in a row is an AI tell regardless of content

When in doubt, merge. The voice does not live in the short sentence.
It lives in the specific word choice and the specific detail of the
longer one.

---

### Rule 3 — Lines should be approximately 8-25 words total, averaging around 10

This is a game played in real time. Dialog appears as a speech bubble
while the player is thinking about their next move. A line under 8 words
barely registers. A line over 25 words competes with the game for
attention and will be skipped.

**Count your words. The target average across a trigger's 10 lines is
10 words per line.** If you calculate the average and it exceeds 15,
cut it down before committing. This is not a vibe check — it requires
arithmetic. Add up the words in all 10 lines and divide by 10. If the
number is over 15, the lines are too long and must be compressed.

The most common cause of inflated averages is writing in a formal or
elaborate register and then not compensating elsewhere. Dropping
contractions alone ("I have" instead of "I've," "I do not" instead of
"I don't") adds 1-2 words to every sentence. Stacking enthusiasm
modifiers ("with great anticipation," "most wonderfully," "tremendously
gratifying") adds 3-4 more. After two such choices per line across 10
lines, you can easily hit an average of 17-18 words without noticing.

**The formal register trap:** If a character speaks formally — no
contractions, aristocratic vocabulary, elaborate clauses — you must
find length elsewhere to compensate. Use shorter constructions in the
*other* part of the sentence. Cut modifier tails. Let some sentences
stand at 8-9 words. The formal register should sound formal, not long.
The voice survives compression. The elaborate phrasing does not.

**The Sweeper exception note:** The Sweeper's existing lines are
intentionally long as a character trait (methodical, can't stop
elaborating). This is a documented exception for that one character,
not a license for others.

---

### Rule 3 — Characters react, they do not commentate

A commentator describes what happened on the board. A character reacts
to what it means to them.

**Commentator (wrong):**
> "You made a strong move that shifted the position in your favour."

**Character (right):**
> "You found something I hadn't accounted for. I don't enjoy that."

The player already knows a good move was played — the trigger fired
because of it. The line should not confirm information the player has.
It should reveal something about the character.

---

### Rule 4 — No information the character cannot have

The character has no access to the chess engine's analysis. They do
not know:
- Which piece moved
- Which square was vacated or occupied
- Whether a capture occurred
- What piece type was captured
- What the eval score is
- Which variant mechanic was just used (unless the trigger specifies it)
- Whether the opponent is in check (unless it's a check-specific trigger)
- How many pieces remain on the board

These facts are not available to the character. Lines that state them
are factually wrong at runtime and will produce nonsense when the
trigger fires in a context where the assumed event did not occur.

**This applies to generic triggers especially.** `PLAYER_GOOD_MOVE`
fires on any sufficiently good move — promotion, capture, quiet
manoeuvre, or tactical setup. A line that says "nice capture" will
fire when the good move was a quiet king shuffle. Write for the
situation, not the specific event.

---

### Rule 5 — Every line must sound like a human being said it

Read every line aloud and ask two questions: does this sound like a human
being? And does this sound like this specific human being?

If it sounds like a chess analysis tool, a game tutorial, or a chatbot
summarising the situation, rewrite it. Specific tells to eliminate are
listed in Part 3.

The character's voice lives in small spoken words: "honestly," "frankly,"
"actually," "really," "well," "oh," "look." These are words real people
use when talking, not writing. They signal presence, opinion, and
personality in a way that formal prose never does. One well-placed
"honestly" can do more for a line's warmth than rewriting the whole thing.

Spoken rhythm also comes from punctuation. A comma before "and" or "but"
is not optional decoration — it is where a voice pauses. Missing commas
produce lines that read as written text, not speech. Every conjunction
joining two clauses needs a comma. Every relative clause introduced by
"which" after a natural pause needs a comma. Read the line aloud:
wherever you breathe, there should be a comma.

---

### Rule 6 — Don't pigeonhole the character into one talking point

A character is defined by their mechanic and their personality, but
they have a life outside of both. Not every line needs to reference
the variant's hook. A character who mentions their theme in every
single line becomes a parody of themselves.

Variety is what makes a character feel alive. Within a trigger's
10 lines, some should come from the theme, some from the character's
attitude, some from their history or relationships, some from their
specific obsessions. The theme is the flavour — it should not be the
only ingredient.

---

### Rule 7 — Don't repeat the same grammatical pattern across all 10 lines

When every line in a trigger follows the same structure — one-word
reaction followed by a short comment, or two short sentences, or a
question followed by an answer — the player notices after a few
encounters. The repetition signals that the lines were generated rather
than written.

Within a trigger's 10 lines, vary the sentence shape. Open with a
full sentence sometimes. Start mid-thought sometimes. Let one line be
a question. Let one be an interruption of itself. The grammar of the
line is part of the character's voice.

---

### Rule 8 — Don't use the same emotional register for all 10 lines

Ten lines of identical energy is exhausting or numbing. A character
whose every `PLAYER_GOOD_MOVE` line is sarcastic kills the sarcasm
through repetition. Within a trigger, the emotional register should
vary:

- Some lines: dismissive
- Some lines: genuinely rattled
- Some lines: deflecting with humour
- Some lines: revealing something the character didn't mean to reveal
- Some lines: recovering composure too quickly
- Some lines: not recovering at all

This range is what makes a character feel like a person under pressure
rather than a machine outputting a consistent attitude.

---

### Rule 9 — Don't narrate what the player can already see

The board told the player what happened. The dialog should not
re-describe it. A `PLAYER_WINS` line that says "You checkmated me" is
pure narration of a visible fact. A line that says "I don't know when
I lost this — somewhere in the middle I just lost it" tells the player
something the board doesn't.

The game event is the context. The character's reaction to the *meaning*
of the event is the dialog.

---

### Rule 10 — Don't use the character's thematic keyword as a crutch

If a character's theme is fire and 8 of 10 `PLAYER_BAD_MOVE` lines
contain the word "explosion," the mechanic becomes a punchline. The
theme is a flavour — some lines should come from an entirely different
angle: the character's pride, their past, their irritation, their
relationship to the player. Let the theme breathe by sometimes
not using it.

---

### Rule 11 — Every line must be attributable to this specific character

Ask: could this line have been written for any character in a similar
situation? If yes, rewrite it.

*"You found the right move there."* belongs to no one. *"That was
correct. I wish it wasn't."* starts to belong to someone. *"My
grandfather had a phrase for moves like that, and I don't want to
repeat it."* belongs to a specific person with a history.

The fingerprint of the character doesn't have to come from their
theme. It can come from their attitude, their vocabulary, their
specific preoccupations, their way of processing stress. But every
line should feel like it came from *this person* and not from a
default dialog generator.

The concrete test: strip the variant's name and mechanic from every
line in a trigger. Do the remaining lines still sound obviously like
this specific person? If yes, you've written a character. If no, the
lines are thematic placeholders dressed up as a person.

The model for this: Papyrus has never tasted his own spaghetti. He
makes it because *"EVERYBODY ELSE LOVES IT."* That detail has nothing
to do with chess puzzles or the Royal Guard. It is entirely specific
to him, slightly embarrassing, and costs him something to have be
true. That is what attribution means.

---

### Rule 12 — Don't explain the observation — let it land

Many lines over-explain: *"You blocked my diagonal. I respect that
you can see diagonals."* The second sentence explains the first.
*"That move had real tactical depth. I'm impressed by your
calculation."* The second sentence is a label for the first.

Trust the player to understand the implication of a well-written
line. Cut the explanatory tail. If the first sentence lands, the
second is noise.

---

### Rule 13 — Characters can be wrong, biased, and inconsistent

Real people misread situations. They project their preoccupations
onto events. They sometimes dismiss a strong move because they're
too proud, or panic at a mediocre one because they're rattled.

A character who always correctly assesses the board is a commentator.
A character who sometimes confidently misreads the situation — that's
a person. Lean into bias. Let pride speak before judgement.

The most powerful version of this: the thing the character is wrong
about is **load-bearing** — it organises their whole self-image. This
isn't a casual mistake or a flaw to be corrected mid-story. It's the
belief that holds the character together, the one they're not quite
able to examine. Write toward that. A character whose PLAYER_WINS
lines reveal — without the character realising — that they always
assumed they were better than this, that they weren't prepared for
the possibility of being genuinely outplayed: that is a person, not
a type. The wrong belief doesn't need to be named. It just needs to
show through.

---

### Rule 14 — Avoid hollow character-summary language

These phrases are placeholders where a real line should go:

- *"as always"* / *"as I always do"*
- *"as is my way"* / *"this is what I do"*
- *"naturally"* / *"of course"*
- *"as expected"* / *"as it must"*
- *"it was always going to be this way"*
- *"this is the way"* / *"this is how it works"*

These phrases describe the character from the outside. They tell the
player what kind of character this is rather than being that character.
They signal that the writer ran out of specific things to say and
reached for a summary.

Replace them with something specific. What does the character actually
think right now? What specific detail from their life or personality
applies here?

---

### Rule 15 — OPPONENT_WINS and PLAYER_WINS must not be flat

A character's 10 winning lines should not all be triumphant. A
character's 10 losing lines should not all be gracious. Both sets
should have range:

**Winning:** arrogance, relief, surprise, satisfaction, a moment of
genuine respect for the opponent, deflection from almost losing.

**Losing:** bitterness, introspection, deflection, a moment of genuine
acknowledgment, denial, something the character reveals about
themselves they didn't mean to.

If all 10 winning lines are confident and all 10 losing lines are
composed, the character has no texture.

---

### Rule 16 — Lines should occasionally reveal life outside this game

Characters who only ever reference chess — even through their specific
theme — feel paper-thin. A line that implies the character has a
history, a relationship, a job, a worry, or an opinion about something
else entirely makes them feel like a real person.

The implication does not need to be explained. *"My brother would
have seen that move immediately. I hate that about him."* tells the
player something about the character without explaining it. It lands
because it's specific and because it costs the character something
to say it.

---

### Rule 17 — Characters should not know they are in a game

Lines must not break the fiction. Phrases that imply the character
is narrating or performing for an audience should be cut:

- *"that's the whole skill"*
- *"that's the whole point of [variant name]"*
- *"this is what this board does"*
- *"that's how this game works"*

The character is playing chess. They are not explaining the rules.
They are not aware that the player is reading their dialog. They
speak as someone in a real encounter — not as a tutorial voice,
not as a sports commentator, not as a game design document.

---

### Rule 18 — Crack the armour occasionally

A character who is uniformly confident, uniformly composed, or
uniformly consistent is wearing a mask. Real people contradict
themselves. Pride slips. Composure fails for one line, then returns
too quickly.

The most memorable lines in any character set are usually the ones
where the mask drops briefly. One line of genuine vulnerability in a
confident character lands harder than ten lines of armour.
It does not "break character" — that crack is the character.

---

### Rule 19 — Each character has a register; write inside it

Every character has a specific emotional register — their default
temperature, vocabulary, and way of processing the world. Before
writing a single line, be able to answer: how does this character
talk when they're pleased? When they're rattled? When they're
trying to seem unbothered?

The register is not a list of topics. It is a voice. The Tinyhome
Owner says "honestly" and "frankly." They say "which I enjoy explaining
to people." They say "Oh, the blog is going to be very good this week"
not "I will document this outcome on my blog." The difference between
those two lines is the entire difference between a character and a
description of a character.

Write the line as the character would say it out loud to someone
standing in front of them, not as a writer describing what the
character thinks.

---

## Part 3 — AI Writing Tells

These are specific words, phrases, and patterns that mark text as
AI-generated. They are common in ChatGPT, Claude, and similar tools.
Scrub all of them from every dialog line before committing.

This list is not exhaustive. The goal is to train your eye for the
underlying pattern: AI writing tends toward **summarising**, **labelling**,
**hedging**, and **explaining** rather than **being**.

### Hollow affirmations and transitions
| Remove | Replace with |
|---|---|
| *"Fascinating."* | Something specific about what fascinates them |
| *"Interesting."* (alone) | The actual reaction |
| *"Indeed."* | Cut or rewrite |
| *"Certainly."* | Cut or rewrite |
| *"Absolutely."* | Cut or rewrite |
| *"Fair enough."* | More specific concession |
| *"Fair point."* | More specific concession |
| *"Well played."* (alone) | Something specific about why |
| *"Impressive."* (alone) | Something specific about what impresses |
| *"Noted."* (alone) | Cut or follow with something real |

### Summary phrases that describe the character instead of being them
| Remove | Replace with |
|---|---|
| *"as always"* | Specific detail |
| *"as is my way"* | Cut |
| *"naturally"* | Cut |
| *"of course"* | Cut |
| *"as expected"* | Cut or rewrite |
| *"as it must"* | Cut |
| *"this is what I do"* | Show it instead |
| *"it was always going to end this way"* | Rewrite — too omniscient |
| *"that is the nature of [thing]"* | Cut |
| *"that is how [thing] works"* | Cut |

### Labelling instead of reacting
| Remove | Replace with |
|---|---|
| *"A [adjective] move."* as the entire line | The reaction to it |
| *"That was [adjective]."* as the entire line | Why it matters to the character |
| *"Clean."* / *"Solid."* / *"Sharp."* alone | The character's response |
| *"A tactical [noun]."* | Cut the label entirely |
| *"A well-executed [noun]."* | Same |

### Chess analysis language
| Remove | Replace with |
|---|---|
| *"the position"* (as a clinical noun) | Something more personal |
| *"the evaluation"* | Cut |
| *"in my favour"* / *"in your favour"* | Rewrite through the character's experience |
| *"tempo"* (unless the character is a chess pedant by design) | Find the human equivalent |
| *"material"* (in most contexts) | Find the human equivalent |
| *"continuation"* | What it means to the character |

### Passive hedging
| Remove | Replace with |
|---|---|
| *"Perhaps..."* as an opening | More specific doubt |
| *"I suppose..."* | More specific concession |
| *"It seems..."* | More specific observation |
| *"It appears..."* | Same |
| *"One might say..."* | The character says it directly |

### Structural AI patterns to catch in review

**The em-dash as a crutch:** The em-dash is one of the clearest signals
of AI-generated prose. It appears constantly in LLM output, used to bolt
a second clause onto a sentence that would be stronger without it.
In almost every case the em-dash is papering over a sentence that hasn't
found its own ending yet. Em-dashes are forbidden in dialog. If you find
yourself reaching for one, either end the sentence where the dash would
be, or restructure so both halves become one clean sentence without the
seam showing.

**The three-sentence staccato:** The single most recognisable AI dialog
pattern. Three short declarative sentences in a row. *"Good. Not what I
needed. I'll acknowledge it."* This rhythm is baked into LLM output and
is almost never how a real person speaks. Maximum two sentences per line,
always.

**The metronome:** All 10 lines in a trigger are approximately the same
length. Real speech has violent variation. The target average is around
10 words per line. Some lines should be 8 words, some 16, and the rhythm
of a trigger set should be jagged, not uniform. Pushing every line to the
20-25 word ceiling is as much an AI tell as writing them all at 5 words.
If the average across a trigger exceeds 15 words, rewrite at least half
of the lines to be shorter.

**Enthusiasm modifier stacking:** A specific subtype of the metronome
that appears when writing a character with an elaborate or expressive
register. The pattern: every line gets a modifier tail — "which I find
tremendously gratifying," "with great anticipation," "most wonderfully,"
"I find that most satisfying." Each individual modifier seems harmless.
Across 10 lines, they read as a generated formula. The character's
enthusiasm should come from the specific content of what they say, not
from affixing an enthusiastic phrase to the end of every sentence. Cut
the modifier tails and let the line land on its own. One or two genuine
enthusiasm markers per trigger is a voice. Ten of them is a tic.

**The treadmill effect:** Restating the same idea in the second sentence
that was just stated in the first. *"I had this rated as manageable. The
rating was wrong."* The second sentence adds nothing the first didn't
already imply. Cut it. If a line has two sentences, the second sentence
must earn its place by doing something different — extending, contrasting,
or landing — not labelling what came before.

**The abstraction trap:** AI uses vague language where a specific human
detail would land harder. *"The board creates a distinctive kind of game"*
versus *"Three moves in and you're already in each other's business — that
doesn't happen on a standard board."* The specific detail always wins.
When a line feels thin, ask: what specific thing would this character
actually think about right now?

**The two-sentence formula:** Observation + label for that observation.
*"You found the right move. That was the correct instinct."* The second
sentence adds nothing. Cut it.

**The chiasm:** Reversing a phrase to sound profound. *"The board changes.
The player changes with it."* This sounds like something but says nothing.
Cut it.

**Hollow affirmation followed by pivot:** *"Good. But..."* / *"Solid.
However..."* / *"Well played. Unfortunately..."* Real people do this too,
but AI does it in every single line. If it appears more than once across
a trigger's 10 lines, rewrite.

**The list-of-three:** AI frequently produces lines with three parallel
items. *"Cool. Calculated. Correct."* / *"My pieces, my plan, my win."*
These read as AI prose rhythms, not speech. Use sparingly and only where
the character's voice genuinely speaks in rhythms.

**Latinate bias:** AI defaults to longer, Latin-derived words over shorter
Anglo-Saxon ones. *"Utilise"* not *"use."* *"Commence"* not *"start."*
*"Facilitate"* not *"help."* *"Demonstrate"* not *"show."* In dialog,
the shorter word is almost always correct. If a word has a common two-
syllable version, use it.

**Cosmic/abstract summary statements:** *"Everything is [theme]."*
*"The [theme] never lies."* *"In the end, [theme] wins."* These are
the AI's way of summing up a theme. They sound like fortune cookies.
They are not dialog. Delete them and write the specific thought
the character is actually having.

**Uniform positivity:** RLHF training makes AI writing relentlessly
upbeat and gracious. Every challenge is an opportunity. Every loss is
a lesson. Every acknowledgment of the opponent comes with a silver lining.
Real characters are sometimes just annoyed, or hurt, or flat. Not every
loss line needs to have composure. Not every win line needs to be magnanimous.

**The tacked-on acknowledgement:** A line ends with a short clause that
performs self-awareness without earning it. *"...and I recognise that."*
*"...and I have to reckon with that."* *"...and I acknowledge that."*
*"...and that's on me."* These are AI gesturing at humility. They add
nothing because they name the emotional beat instead of landing it. If
the line has done its job, the acknowledgement is redundant. If the line
hasn't done its job, the acknowledgement can't save it. Cut the tail.
The line ends where the real content ends.

---

## Part 4 — Writing Characters Based on Real Source Material

Some characters in this project are not invented from scratch. Their
personality, vocabulary, speech rhythms, and preoccupations are drawn
from a specific real person, fictional character, or cultural archetype.
The Groundskeeper, for example, is built on Hank Hill from *King of the
Hill*. When a character has a source like this, the design document will
say so explicitly.

Writing these characters requires an additional layer of discipline. The
rules in Parts 1–3 still apply in full. But there is a specific failure
mode that only affects sourced characters, and it is the most common one:

**Rendering the source instead of being the source.**

This is the difference between a writer describing Hank Hill and Hank
Hill speaking. A line that *captures the character's themes* is not the
same as a line *written in the character's voice*. Sourced characters
fail when the lines are accurate about the character in the third person
— correct about their obsessions, their history, their relationships —
but the underlying prose sounds like a competent writer rather than the
person themselves.

The rules below are how to avoid that.

---

### Rule 20 — Identify the source's actual speech patterns before writing a word

Before writing any line for a sourced character, you need to be able to
answer these questions specifically:

- **Sentence length.** Does this person speak in long sentences or short
  ones? Do they use subordinate clauses, or do they speak in
  main clauses only?
- **Vocabulary level.** Do they use common words or elevated ones?
  Do they use technical language from their domain?
- **Rhetorical devices.** Do they use metaphor? Irony? Understatement?
  Do they ever use rhetorical questions, or do they only make
  statements?
- **Signature constructions.** Does this person have specific phrases,
  tics, or sentence openings they return to? What do they sound like
  when they are pleased? When they are rattled? When they are trying to
  seem unbothered?
- **What they do not do.** This is as important as what they do. A
  character who never uses irony must never produce an ironic line. A
  character who speaks in flat declaratives must never produce an
  elegantly constructed clause.

Do not proceed until you can answer all of these from the source
material, not from a general impression of the character.

---

### Rule 21 — The voice lives in the sentence structure, not the subject matter

The most common mistake when writing a sourced character is correctly
identifying *what they would talk about* while missing *how they would
say it*.

A line can reference all the right topics — the correct obsessions, the
correct preoccupations, the correct emotional beat — and still sound
nothing like the source, because the underlying prose is constructed
the way a writer constructs prose rather than the way the character
speaks.

**Example of the failure:**

Hank Hill's character is defined by his quiet territorial conviction
about the Hill, his propane opinions, and his worry about his son.
A line like *"The Hill recognises a king when it sees one, though I plan
to challenge that"* hits all three contextual notes: it's territorial,
it's about the Hill, it has implied confrontation. But it uses rhetorical
personification ("the Hill recognises"), an elegant pivot in the second
clause ("though I plan to challenge that"), and a general air of literary
construction. Hank Hill does not speak this way.

**The same moment, written in his voice:**

*"Your king is on the Hill, and I'm going to move it off there."*

Flat. Declarative. No ornament. That is the voice.

The test: strip all proper nouns from the line and ask whether the
underlying sentence structure sounds like the source. If the answer
is no, the structure is wrong regardless of the content.

---

### Rule 22 — Sourced characters have things they never do

Every real speaker has constructions they simply do not use. These
prohibitions are as important to the character's voice as their
positive traits, and they are easier to violate because they are
absences rather than presences.

Identify at least three things the source character **never does** and
hold to them across every line:

- Do they avoid rhetorical flourish? Then no personification, no
  chiasm, no elegant callback in the second clause.
- Do they avoid hedging language? Then no "perhaps," no "I suppose,"
  no "I think that might be."
- Do they avoid abstraction? Then every line must land on a specific,
  concrete thing — no statements about categories or general
  principles unless immediately grounded in a specific instance.
- Do they avoid irony? Then every line is meant literally. No
  implication that the character is performing an emotion they don't
  have.

These prohibitions apply even when violating them would produce a better
line by conventional writing standards. A clever line that the character
would never say is wrong. A plain line they would actually say is right.

---

### Rule 23 — Signature phrases are sparingly placed, precisely timed moments

Most sourced characters have one or more signature phrases — verbal tics,
recurring constructions, or specific words that are strongly associated
with them. These are powerful when used correctly and quickly become
parody when overused.

**The correct usage:**

A signature phrase marks a moment of genuine need — a specific
emotional or situational beat that the phrase was developed to handle.
Hank Hill's "I tell you what" is not a verbal filler. In the show it
fires when he is genuinely recalibrating: encountering something he
hadn't expected, arriving at a conclusion he didn't anticipate, or
settling himself before saying something that matters. It costs him
something to say. That is when it belongs in a line.

**The failure modes:**

- **As filler:** placing the phrase at the start of a line because the
  line needed an opener, not because the moment genuinely called for it.
- **As flavour seasoning:** distributing it evenly across triggers so
  every trigger has "one Hank Hill moment," treating it as an ingredient
  rather than a beat.
- **As a crutch:** using the phrase to carry lines that have no other
  character content, so the signature phrase is doing all the work of
  making the line feel like the source.

The rule: **one signature phrase instance per trigger, maximum.** Its
placement within the trigger should be the moment of greatest genuine
recalibration — not necessarily the first line, not necessarily the last,
but wherever the emotional beat is strongest. If no line in the trigger
genuinely calls for it, omit it from that trigger entirely rather than
forcing a placement.

---

### Rule 24 — Off-topic obsessions must arrive naturally, not be inserted

Characters based on real source material often have obsessions that
are genuinely irrelevant to the game being played. Hank Hill has
propane. These details are some of the most powerful lines in a
character's set when they land correctly, and some of the most
damaging when they don't.

The failure mode is insertion: deciding that a trigger needs a propane
line and finding a place to put one. The result is a line where the
connection between the chess moment and the off-topic detail is
visible — the writer reaching for it — rather than the character's
mind simply drifting there the way it actually would.

**Insertion (wrong):**

*"The Hill's occupied correctly, and the propane in the corner is
keeping the equipment warm."*

The propane is bolted on. There is no reason this moment would make
him think about propane. The connection is entirely manufactured.

**Natural arrival (right):**

*"I was thinking about propane for part of that game, which I'll
acknowledge is not ideal."*

The propane arrived because his mind drifted during a win. He notices
it himself. He's slightly embarrassed by it. The detail reveals
something — he couldn't stay fully focused — rather than just
decorating a line.

The test: can you explain *why* this character's mind went to this
subject *at this specific moment*? If the honest answer is "because
I needed a propane line in this trigger," the insertion is showing.
Find the moment where the drift is genuinely motivated and place it
there. If no such moment exists in a trigger, the obsession does not
belong in that trigger.

---

### Rule 25 — The character's blind spots and failures are part of the voice

Sourced characters from real material are not self-aware about
everything. Hank Hill does not know that his propane opinions function
as psychological warfare. He does not know that his quiet worry about
his son is visible to everyone. He does not know that his steadfast
play forces errors in opponents who are better at chess than he is.

These blind spots are not decoration — they are load-bearing parts of
the voice. A line that implies self-awareness the source character does
not have breaks the fiction more severely than a knowledge-rule
violation, because it changes *who the person is*.

Before committing any line, ask: **does this character know this about
themselves?** If the answer is no, the line cannot be written from
their perspective. It can only be *about* them, written from the
outside — which is wrong.

The corollary: lines where the character reveals something they *don't
know they're revealing* are among the strongest in any set. The worry
about his son that sits in every third line without ever being directly
named. The brief acknowledgment of discomfort during a win that he
wraps up too quickly. These land because the character didn't mean to
say them. Write toward those moments deliberately.

---

### The Sourced Character Self-Review

Run these checks in addition to the standard checklist in Part 5.

- [ ] Have you identified the source's sentence structure, vocabulary level,
  and rhetorical habits from the actual source material?
- [ ] Have you listed at least three things the source character never does,
  and checked every line against that list?
- [ ] Does every line pass the structure test — does the underlying sentence
  construction sound like the source, not just the subject matter?
- [ ] Is each signature phrase placed at a moment of genuine emotional need,
  not distributed evenly as flavour?
- [ ] Is the signature phrase count one per trigger or fewer?
- [ ] Do any off-topic obsession lines feel inserted? Can you explain why the
  character's mind went there at that specific moment?
- [ ] Does any line imply self-awareness the source character does not have?
- [ ] Are there any lines where the character reveals something they didn't
  mean to reveal?

---

## Part 5 — The Self-Review Checklist

After writing a batch of dialog — before committing — go through this
checklist for every trigger. If the answer to any item is "no," rewrite
before continuing.

### Before you write anything
- [ ] Have you read the character's design document in `chess-variants-app/docs/character-designs/`?
- [ ] If the character is sourced from real material, have you completed the Sourced Character Self-Review in Part 4?

### Knowledge rules
- [ ] Does every line stay within the trigger's knowledge column?
- [ ] Are there any lines that name a piece type that was moved or captured?
- [ ] Are there any lines that reference a specific square?
- [ ] Are there any lines that reference whether a capture occurred (unless the trigger guarantees it)?
- [ ] Are there any lines that reference variant-specific mechanics in a generic trigger (PLAYER_GOOD_MOVE, PLAYER_BAD_MOVE)?

### Length and variety
- [ ] Are all lines between 8 and 25 words? (Exceptions must be intentional and match a documented character trait.)
- [ ] Is every line 1-2 sentences? Are there any lines with 3 or more sentences?
- [ ] Does every sentence in every line contain at least 5 words? (This is the single most common failure. Check every sentence individually, not just every line.)
- [ ] Are there any two consecutive short sentences in the same line?
- [ ] **Count the words.** Add up the words across all 10 lines and divide by 10. Is the average below 15? If the average is above 15, the lines are too long — compress before continuing. (Target is 10. Up to 13 is acceptable for characters with formal or elaborate registers. Above 15 requires a rewrite.)
- [ ] Does any line contain an em-dash? (Forbidden. Restructure the sentence.)
- [ ] Do the 10 lines vary in sentence structure, not repeating the same grammatical skeleton?
- [ ] Do the 10 lines cover a range of emotional registers, not a single consistent tone?

### Character voice
- [ ] Could any line be said by a different character without change? (If yes, rewrite it.)
- [ ] Is the character's theme referenced in every single line? (If yes, add variety.)
- [ ] Is there at least one line that reveals something about the character beyond the variant's mechanic?
- [ ] Are there any lines where the character's composure or armour briefly fails?
- [ ] Can you name this character's central obsession — the specific thing they care about that has nothing to do with chess? Is it present, at least implicitly, in at least one line of this trigger?
- [ ] Are the lines built from that specific obsession, or from a generic trait? ("Confident" is a trait. "The blog," "propane," "the imaginary flame store" are obsessions.)
- [ ] What is the character wrong about — the load-bearing belief that organises their self-image? Does at least one line in `OPPONENT_WINS` or `PLAYER_WINS` let that belief show without the character noticing?

### Writing quality
- [ ] Read every line aloud. Does it sound like a human being?
- [ ] Does it sound like this specific character, not just any character in this situation?
- [ ] Does every conjunction joining two clauses have a comma before it?
- [ ] Does every "which" clause that follows a natural pause have a comma before it?
- [ ] Are there any lines that would benefit from a small spoken word ("honestly," "frankly," "actually," "well," "oh") to add warmth or presence?
- [ ] Are there any AI tells from the list in Part 3?
- [ ] Are there any lines that explain the observation rather than landing it?
- [ ] Are there any summary phrases ("as always," "naturally," "as expected")?
- [ ] Are there any lines that narrate what the player can already see on the board?
- [ ] Are there any lines that describe the character from the outside rather than being the character from the inside?
- [ ] Are there any lines that break the fiction (character aware of being in a game, narrating rules)?

### OPPONENT_WINS / PLAYER_WINS specific
- [ ] Is the winning set varied — not all triumphant?
- [ ] Is the losing set varied — not all gracious?
- [ ] Do both sets include at least one line that surprises?

---

## A Note on the Existing Dialog

Some characters in `CharacterCatalog.kt` were written before these
guidelines existed. Violations of these rules exist in the current
dialog and are known. The rules apply to all new dialog and to any
dialog that is substantially revised. Do not "fix" existing lines
unless you are rewriting the whole trigger set for a character — partial
fixes applied to a subset of lines create inconsistency within the same
trigger. When you do rewrite, rewrite all 10 lines together so the set
has a consistent quality and emotional range.
