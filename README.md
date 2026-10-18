# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

TakeMeter sorts comments from [r/Games](https://www.reddit.com/r/Games/), a
general video-game news subreddit, by **how they argue**, not by whether they're
right. A `backed` comment supports its point with something you could go and
check (a sales figure, a date, a quote, a link, a specific thing that
happened). A `bare_take` states a verdict or prediction with nothing behind
it. A `reaction` is a joke, a feeling or a jab with no claim to argue. Give it
a comment and it returns one of those three labels, from a DistilBERT model
fine-tuned on 230 r/Games comments collected and labeled for this project
(pre-labeled by Claude; see How I Used AI).


---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

Three labels, about how a comment argues rather than whether it's "good".

### `backed`

**Definition:** Makes a claim and supports it with at least one stated fact a
reader could check (a number, date, price, sales figure, quote, link, or a
specific thing that happened), offered as evidence rather than as the topic.

**Example 1:**
> This gets repeated all the time, but most of people who had any say in how the game actually shaped out is still there. Even John Gonzalez is back. Just go to mobygames and start going through the credits, most of the staff with "lead" on their title are still there.

**Example 2:**
> This isn't the first time it's been said though. There was a leak probably 4 months ago suggesting a February 2027 launch for PC that didn't get much traction. My question is whether they mean fiscal Q1 or calendar Q1. If it's calendar Q1 then it lines up because the leak suggested take two wanted the PC launch before the end of their fiscal year.

### `bare_take`

**Definition:** States an opinion, verdict or prediction as if it were true,
with no checkable fact offered in support. This includes sarcasm whose point is
a clear verdict.

**Example 1:**
> There is nothing worth exploring. Starfield has the most boring Sci-Fi setting in fiction to date. It is the epilogue of a setting where everything is irrelevant and the interesting events already happened. You just go around a santized, corporate setting with nothing noteworthy about it.
>
> What is the point of writing more quests or adding more hand-made exploration in such a crap setting?

**Example 2:**
> This is a preview of things to come whenever Sony foolishly decides to release the PS6 before the end of the decade.

### `reaction`

**Definition:** Expresses a feeling, a joke or a jab without asserting any
claim about a game, company or the industry that someone could argue against.

**Example 1:**
> Looks like it won't be getting the @CanYouDismemberTheDog endorsement

**Example 2:**
> Meanwhile, I want a Subaru with Subaru on it

### The hardest boundary

**Which two labels:** `backed` vs `bare_take`.

**The decision rule I used every time:** Ask: *does the comment state a fact
someone could look up or verify, and use it as evidence for its point?* If
yes, it's `backed`, whatever the tone, even sarcastic or angry. If no, it's
`bare_take`. Three things that do **not** make a comment `backed`:
- **Naming a game, person or event without saying what happened.** "Fallout
  76 showed us that already" names evidence but doesn't state it.
- **A number that's the topic, not the support.** "Paying $70 for this is
  insane" judges the price; the $70 isn't evidence for anything.
- **The poster's own vague experience or a sweeping generalisation.** "after
  1000s of hours", "every live-service game this year flopped".

A specific first-hand observation does count ("the frame rate dropped under 30
in the first town"), because someone else could check it.

**The borderline post that forced this rule:**
> Think what really killed starfield more than anything was lack of meaningful exploration. I mean I can go explore fallout or Skyrim right now and potentially find something I havent found before even after 1000s of hours. Starfield you're more than likely just going to find the same poi that you found on the previous planet and the one before that and the one before that.

I asked a separate Claude agent to make the strongest case for each label. The
case for `backed`: it names Fallout and Skyrim as a comparison and makes a
claim you could test by landing on two planets. The case for `bare_take`:
"killed" is never shown, "1000s of hours" is unmeasurable, and with the game
names removed it's "Starfield's exploration is repetitive", which is opinion.
Under the rule it's **`bare_take`**. The games are named as a comparison, not
as stated facts, and the only number is the poster's own vague experience.

**The second boundary,** `bare_take` vs `reaction`: if you strip out the
joke or the feeling and an arguable claim about a game, company or the industry
is left, it's `bare_take`. If nothing arguable is left, it's `reaction`.
"Lmao Ubisoft announcing another open-world game with a map full of icons.
Shocking." leaves "Ubisoft keeps making the same game", so it's `bare_take`.

**Stress test.** Before labeling anything, I had a Claude agent write 8
comments sitting on these boundaries without telling me their labels. Under my
first definitions I couldn't call 6 of the 8 cleanly. The problems were sarcasm
with a number in it, games named as "evidence", and a price that was the topic.
That's where the three "does not count" lines and the sarcasm clause came
from. With them, all 8 sort in one read: #1, #2, #3, #4, #5 and #8 are
`bare_take`, and #6 ("sells 20 million copies by default. Incredible.") and #7
(the frame-rate drop) are `backed`. None of the eight ended up `reaction`.
That's a hint that `reaction` will be my thinnest label, so I need to watch it
in the counts.

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Public comments from
[r/Games](https://www.reddit.com/r/Games/), a general video-game news and
discussion subreddit, not tied to one game or release. I collected 1,084
comments from 16 threads: the top threads of the past week and month plus the
current hot list, each with at least 60 comments. The topics were GTA VI's
trailer and PC date, Starfield 2, the Forza Horizon 6 ban wave, NBA 2K25 being
delisted, ARC Raiders going PvE, the RTX 5090, and others. Comments were read
with [rdt-cli](https://github.com/jackwener/rdt-cli), the Reddit backend the
[Agent Reach](https://github.com/Panniantong/Agent-Reach) project uses, logged
in as my own account and read-only, with a few seconds between requests.
Reddit blocks anonymous scripted requests, which is why a logged-in tool was
needed.

**Notes from reading (Milestone 1).** I read 40 randomly sampled comments
before deciding anything. Distinctions that actually showed up in the text:

1. **Some back their point with something you could go and check.** "There
   was a leak probably 4 months ago suggesting a February 2027 launch for
   PC… My question is whether they mean fiscal Q1 or calendar Q1." / "Just go
   to mobygames and start going through the credits, most of the staff with
   'lead' on their title are still there." / A list of the specific games
   Debra Wilson's characters were modelled on (Jedi Survivor, Suicide Squad,
   Forspoken, Death Stranding 2).
2. **Many state a verdict with nothing behind it.** "Starfield has the most
   boring Sci-Fi setting in fiction to date." / "This is a preview of things to
   come whenever Sony foolishly decides to release the PS6 before the end of
   the decade." / "Nothing new, absolutely unacceptable."
3. **Many are pure feeling, a joke or a jab, with no claim to argue.** "Looks
   like it won't be getting the @CanYouDismemberTheDog endorsement" / "Meanwhile,
   I want a Subaru with Subaru on it" / "it is really an accomplishment how so
   many whiny obnoxious people managed to find the same subreddit together".
4. **Some are personal stories**, like a magical moment in the WoW beta, or
   learning baseball rules from sports games as a kid. They're concrete, but
   most aren't arguing for anything.

**How I labelled them:** All 230 rows were **pre-labeled by Claude**, and then
**I checked rows 1–158 myself** and agreed with every one of those labels.
Rows 159–230 were not hand-checked. The `note` column says which is which
(`pre-labeled by Claude, checked by me (agreed)` or `pre-labeled by Claude,
not hand-checked`).

From the 1,084 collected comments, 230 were drawn evenly across the 16
threads and across short (<120 characters), medium and long (>350) comments,
after removing exact duplicates, deleted comments, AutoModerator, and comments
that were only a quote or only a link. Claude read each one in full and applied
the definitions and decision rule above, in four batches of about 58. 19 posts
made it hesitate, and those are marked `hard:` in the note with the reason.
Posts are kept whole (the notebook truncates long ones to 128 tokens, not me).
The file is one unsplit CSV; the notebook does the 70/15/15 split.

The unit 6 agreement check (`agreement.py`, criterion 4) will test my
labeling against staff labels on posts I haven't seen labeled.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `bare_take` | 103 | 44.8% |
| `backed` | 75 | 32.6% |
| `reaction` | 52 | 22.6% |
| **Total** | 230 | 100% |

No label is near the 70% cap. `bare_take` is the biggest, as the reading
predicted. `reaction` came out at 22.6%, a little above the ~20% I assumed when
writing criterion 2.

**Three hard cases**

**1.**
> *The post:* "Bobby Kotick who was best mates with Epstein and Maxwell? That Bobby Kotick?"
>
> *Could have been:* `reaction` (it reads as a sarcastic jab) or `backed`.
>
> *I chose, because:* **`backed`**. The jab rests entirely on a stated factual
> claim that anyone could look up (that Kotick was close to Epstein and
> Maxwell), and that claim is the whole case against him. Whether it's true
> isn't the label's job; it's checkable. My rule says tone never
> decides, so a sarcastic comment with a checkable fact as its evidence is
> still `backed`.

**2.**
> *The post:* "“Even with the size of the team that we had for Starfield, it wasn’t big enough to create the amount of content that the grand idea of the game really required,” he explained. “Like, we needed way more.” — I really dont think this was the games problem. Yes it needed more "points of interest" generated on a planet. But the fundamental structure was screwed up. They should have made a handful of large handcrafted maps…"
>
> *Could have been:* `backed` (it contains a direct quote, which my
> definition lists as a checkable fact) or `bare_take`.
>
> *I chose, because:* **`bare_take`**. The quote is the *topic*, what the
> poster is disagreeing with, not evidence for their own point. Their own
> argument ("the fundamental structure was screwed up") has no checkable
> support. This is the "a number that's the topic, not the support" exception,
> applied to a quote.

**3.**
> *The post:* "I've always spent over $1500 CAD for my gaming laptops anyway. Done it three times. And I accept I may have to spend over $2000 the next time (2027-28 latest). But consoles on top (exclusives, connection to living room TV, etc) can screw off…"
>
> *Could have been:* `backed` (dollar amounts and dates) or `reaction`.
>
> *I chose, because:* **`reaction`**. The numbers are about the poster's own
> purchases, and they don't support any claim about a game, company or the
> industry. Strip out the personal story and what's left is "consoles can screw
> off", a feeling rather than an argued claim. This is the case most likely to
> confuse the model, because it *looks* like a `backed` post: digits, dollar
> signs, years.

The other 16 hard cases are in `labels.csv` (search the `note` column for
`hard:`). Most of them sit on the same two lines: a fact delivered with
sarcasm (Halo's string of botched releases, #164), and games or people named
without a fact about them (The Witcher 3, Blizzard North).

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`, the notebook's default, fine-tuned
for 3-way classification.

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128
tokens, seed 42. Labels in the notebook: `LABELS = ["backed", "bare_take",
"reaction"]`.

**Anything I changed from the defaults, and why:** Nothing. The only edits to
the notebook were `NAME`/`EMAIL` and the `LABELS` list. With 230 examples and no
results to react to yet, I had no evidence that any setting needed changing.
128 tokens covers most of these comments; the longest ones (up to 1,000
characters) get cut off near the end, which I'll keep in mind next unit.

**Where it ran:** `mps`, the Apple Silicon GPU on my Mac (PyTorch 2.14.1), as
recorded in `results.json`.

**Split sizes:** train 160 · val 35 · test 35 (the notebook's stratified
70/15/15 split).

| Label | Train | Val | Test |
|---|---|---|---|
| `backed` | 53 | 11 | 11 |
| `bare_take` | 71 | 16 | 16 |
| `reaction` | 36 | 8 | 8 |

Every label has at least 8 in the test split, but `reaction` is right at that
edge. With 8 test posts, one prediction moves its F1 a lot, so expect that
label to be the jumpiest across seeds next unit (criterion 2).

**First look (not analysed yet; that's unit 6):** section 5 wrote
`results.json` and `test_split.csv`. On seed 42 the model predicted `bare_take`
for 34 of the 35 test posts: accuracy 0.486, macro F1 0.269, `reaction` F1
0.000. I'm recording that, not tuning for it. My criteria are already
committed, and the diagnosis belongs to next unit.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Pre-labeling disclosure.** The labels in `labels.csv` were pre-labeled with
Claude (Claude Code), working from my definitions and decision rule. I checked
rows 1–158 and agreed with every label; rows 159–230 weren't hand-checked. The
`note` column records this row by row.

I used Claude Code as a tool throughout the project: to collect the comments
with rdt-cli, to help draft and test the definitions and criteria, and to run
the notebook. The two moments below are where using it changed my decisions.

**Moment 1: stress-testing the labels before labeling anything**

- *What I asked for:* As Milestone 2 suggests, I gave a separate Claude agent
  my three definitions and asked it to write 8 r/Games-style comments sitting
  on the boundaries, without telling me their labels. I also asked it to make
  the strongest case for both `backed` and `bare_take` on one real Starfield
  comment.
- *What came back:* 8 comments, and I couldn't call **6 of them** cleanly
  under my first definitions. "Paying $70 for this is insane" has a number, but
  the number is the topic. "Fallout 76 showed us that already" names a game
  but states no fact about it. "Another Call of Duty that sells 20 million
  copies by default. Incredible." is sarcastic but contains a real figure.
- *What I changed:* I added the three "does not make it `backed`" rules
  (naming without stating, a number that's the topic, vague experience), the
  "tone never decides" rule, and a sarcasm clause to `bare_take`. After that,
  all 8 sorted on one read. The two-sided Starfield argument became the
  worked example under my decision rule; it's `bare_take`.

**Moment 2: checking the criteria could be scored by someone else**

- *What I asked for:* With all five criteria drafted, I asked a fresh Claude
  agent, as Milestone 3 suggests, for one reason each target might be too easy
  and one it might be too hard, and whether each could be checked without
  asking me what I meant.
- *What came back:* Plausible reasons both ways for all five, so I kept the
  numbers. It also flagged four wording gaps. Should "0.62" be compared
  rounded? Does a label the model never predicts count as a pass or a fail?
  How do you score a baseline answer it can't parse? How are the "thirds" cut
  in the confidence check?
- *What I changed:* I checked `baseline.py` and found it's a zero-shot
  pipeline that always outputs one of my labels, so there's nothing to parse,
  and I wrote that into criterion 3. I made criterion 1 compare unrounded
  values, made a never-predicted label an explicit fail (F1 = 0) in criterion
  2, and pointed criterion 5 at the notebook's own `confidence_report`
  definition of the thirds. These were committed before any training ran.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
