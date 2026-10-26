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

**Unit 6.** As in unit 5, I used Claude Code as a tool to run the commands:
`baseline.py` (with and without my definitions, and on the seed 7 and 2024
splits) and notebook sections 6 and 7, before and after. I had it skip the
single-seed section 5 so my committed `results.json` and `test_split.csv`
stayed as they were. It also helped me draft the write-up from the result
files, and I read and edited every section. I wrote my prediction down before
any baseline number existed. The decisions were mine: the prediction, the
labels on the staff set, the improvement, and the verdicts.

**Pre-labeling disclosure (staff set).** Claude pre-labeled the 30 posts in
`my_staff_labels.csv` under `data/staff_taxonomy.md`, before either of us had
seen any staff label. I then read all 30 against the staff rules myself and
agreed with every label. The ones I looked at hardest were
sp_014 (rule 1: the 3% is decoration), the four posts built on the writer's
own numbers (sp_002, 005, 007, 010), and sp_021.

**Moment 3: picking the one improvement**

- *What I asked for:* I gave Claude the unit brief and had it work through
  the milestones. After the before run, it read the training logs to work out
  which of the milestone's suggested fixes the evidence pointed at.
- *What came back:* The before-run training loss ended at 1.01–1.04, barely
  under the ln 3 ≈ 1.10 of a model that knows nothing, after only 30
  optimizer steps. It also pointed out that criteria 3 and 5 were technically
  met by a model guessing `bare_take` for almost everything.
- *What I changed:* Two of the milestone's three suggested fixes are
  relabeling and more data, but neither would show up in a model that hasn't
  fit its own training set. So epochs (3 → 10) became my one change, with the
  value fixed before running it. I also wrote the "met by a one-label model" caveat into my
  verdicts for 3 and 5 instead of counting them as wins.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

Both columns are scored on the same 35 posts in `test_split.csv` (seed 42).
Baseline is `facebook/bart-large-mnli` zero-shot, given my definitions from
`label_definitions.txt`. Trained is the unit 5 model from `results.json`.

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.343 | 0.486 | +0.143 |
| Macro F1 | 0.206 | 0.269 | +0.063 |
| F1 — `backed` | 0.500 | 0.167 | -0.333 |
| F1 — `bare_take` | 0.118 | 0.640 | +0.522 |
| F1 — `reaction` | 0.000 | 0.000 | +0.000 |

**What my definitions were worth.** I also ran the baseline with bare label
names (`baseline_results_bare_names.json`): accuracy 0.286, macro F1 0.273. The
definitions bought +0.057 accuracy but cost 0.067 macro F1. With definitions,
the zero-shot model called 33 of 35 posts `backed`. My `backed` definition is
the longest and most specific sentence, and "makes a claim and supports it"
seems to be something almost any argumentative comment entails. The table
above uses the definitions run as the baseline.

The baseline was also scored on the seed 7 and seed 2024 test splits, for the
criterion 3 column of the run log: 0.314 accuracy on both.

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

Written before running `baseline.py`: the trained model will beat the baseline
only on `bare_take` F1, because it learned to guess `bare_take`; the baseline
will win on `reaction` and `backed`, since those are describable in words.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

The prediction was half right. The trained model won only on `bare_take`
(+0.522) and the baseline won on `backed` (+0.333), as predicted. But the
baseline did not win on `reaction`: both models scored 0 there. The gap is not
evidence that the trained model learned my taxonomy. Both models collapsed
onto a single label, the trained one onto `bare_take` and the baseline onto
`backed`, and the trained model's +0.143 is simply the difference between
those two labels' shares of the test split (16/35 vs 11/35, plus one post).
A constant "always `bare_take`" guesser would score 0.457 and beat the
baseline by the same margin. The baseline is also far slower at runtime: it
loads a 1.6 GB model and scores every post against three hypotheses, while the
trained model scores a whole 35-post split in about 0.05 s (the notebook's
`eval_runtime`).

---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

**Device:** `mps` (Apple Silicon GPU, PyTorch 2.14.1), all three seeds, as
recorded in `results_three_seeds_before.json`. Settings unchanged from unit 5:
3 epochs, lr 2e-5, batch 16, max length 128. 35 test posts per seed (11
`backed`, 16 `bare_take`, 8 `reaction`).

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.62 in every seed | 0.486 | 0.486 | 0.457 | MISSED |
| 2. Lowest per-label F1 | ≥ 0.45 in every seed | 0.000 (`reaction`) | 0.000 (`reaction`) | 0.000 (`backed`, `reaction`) | MISSED |
| 3. Trained − baseline accuracy | ≥ +0.10 (seed 42) | +0.143 (0.486 − 0.343) | +0.171 (0.486 − 0.314) | +0.143 (0.457 − 0.314) | MET |
| 4. Agreement with staff set | ≥ 21/30 (0.70) | 30/30 (1.00) | — one labeling, not per seed | — | MET |
| 5. Most-sure − least-sure third accuracy | ≥ 0.15 in every seed | +0.545 (0.727 − 0.182) | +0.273 (0.727 − 0.455) | +0.364 (0.818 − 0.455) | MET |

The full spread, from section 6:

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.486 | 0.486 | 0.457 | 0.029 |
| Macro F1 | 0.269 | 0.269 | 0.209 | 0.060 |
| F1 — `backed` | 0.167 | 0.167 | 0.000 | 0.167 |
| F1 — `bare_take` | 0.640 | 0.640 | 0.627 | 0.013 |
| F1 — `reaction` | 0.000 | 0.000 | 0.000 | 0.000 |

Accuracy moved only 0.029 across seeds, but it's stable for a bad reason: the
model gives nearly the same answer to every post on every split.

### Confusion matrix

Seed 42 (section 7 prints the first seed's; seed 7 is identical, and seed
2024 differs only in that its one `backed` hit also became `bare_take`):

| true \ predicted | backed | bare_take | reaction |
|---|---|---|---|
| **backed** | 1 | 10 | 0 |
| **bare_take** | 0 | 16 | 0 |
| **reaction** | 0 | 8 | 0 |

**My biggest off-diagonal number, and what it means:** 10 real `backed` posts
were called `bare_take` and 0 went the other way. But the `reaction` row is
just as one-sided (8 of 8 to `bare_take`), so this isn't one boundary the model
failed to learn. It never learned any boundary, and it puts 34 of 35 posts in
the majority label.

---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

Verdicts are on the **before** run (the unit 5 settings). The after run has
its own verdicts under The Improvement.

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.62 in all three seeds | MISSED | 0.486 / 0.486 / 0.457. No seed is within 0.13 of the target. |
| 2 | No label F1 below 0.45 | all 9 per-label F1s ≥ 0.45 | MISSED | 6 of the 9 are below 0.45 (`backed` 0.167 / 0.167 / 0.000). `reaction` is 0 in every seed because it's never predicted, which the criterion counts as a fail. Only `bare_take` clears it. |
| 3 | Beats zero-shot baseline | trained − baseline ≥ 0.10 on seed 42 | MET | +0.143 on seed 42, which is what the criterion names. It also holds on seeds 7 and 2024 (+0.171, +0.143). It's met by the letter of the criterion. But a constant `bare_take` guess would also pass, so this criterion as written doesn't separate "learned something" from "picked the majority label" (see Baseline vs. Trained). |
| 4 | Agreement with staff | ≥ 21 of 30 | MET | `agreement.py` reports 30 of 30 (1.00), well above 21. It doesn't depend on the seed, so one number covers all three columns. The target turned out too loose for this set (see Agreement Report). |
| 5 | Confidence separates right from wrong | most-sure − least-sure ≥ 0.15 in all three seeds | MET | +0.545 / +0.273 / +0.364, all at least 0.15. Same caveat as 3: with a one-label model, this measures which *posts* it's surest are `bare_take`, and the most-sure third is mostly real `bare_take` posts. Mean confidence when right is only 0.44 to 0.46, barely above the 0.33 of a uniform guess. |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

**Criterion 1 (accuracy) and criterion 2 (per-label F1) have one cause: the
model never got far enough from its starting point to learn any boundary.
That's a training problem, not a data problem.** How I know:

- **The matrix.** 34 of 35 posts go to `bare_take` on seeds 42 and 7, and all
  35 on seed 2024. `bare_take` recall is 1.0 in every seed and `reaction` is
  never predicted at all. That's the signature of a model sitting on the
  majority label (45% of my data), not of one boundary being confused.
- **The training loss.** It ended at 1.01, 1.02 and 1.04 across the three
  seeds. A model that knows nothing and guesses evenly over three labels has a
  loss of ln 3 ≈ 1.10, so the model barely moved off that. It didn't even fit
  the 160 posts it trained on, and validation accuracy sat at 0.457 or 0.486
  (the majority share) after every epoch.
- **The arithmetic.** 160 training posts ÷ batch 16 × 3 epochs = **30
  optimizer steps** at lr 2e-5. That's very little for a freshly initialized
  classification head. This is the rare case where the diagnosis points at a
  training setting, which is why the improvement below changes epochs and not
  labels.
- **Ruling out the labels.** If my `backed`/`bare_take` line were applied
  inconsistently, I'd expect errors in *both* directions on that pair, and
  some `backed` predictions. Instead there are no predictions of `backed` or
  `reaction` to be wrong in the first place. The agreement check is the
  direct test of my labeling consistency. I matched staff on all 30 posts,
  including all 7 that staff marked as hard, and 3 of those 7 are exactly
  the "does the fact carry the argument?" call that my own `backed`/`bare_take`
  rule is built on (sp_005, sp_010, sp_014). So there's no sign I apply that
  kind of rule two ways, and nothing in my labeling could explain a model that
  predicts one label for everything.

`reaction` being my thinnest label (36 training posts) isn't the cause of the
*before* miss either: `backed` has 53 training posts and was predicted just
once. Thinness matters later, though. See The Improvement, where `reaction`
is the label still failing once training is fixed.

---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 30 / 30 = 100%

My 30 labels are in `my_staff_labels.csv`, made under `data/staff_taxonomy.md`
before I had seen any staff label. They were pre-labeled by Claude, blind, and
then checked by me (see How I Used AI). Staff's labels are in
`data/staff_labels.csv`; the result is in `agreement_results.json`.

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

**None.** `agreement.py` found 0 disagreements, so there's nothing to sort
into piles (a), (b) or (c). Instead, here are the 7 posts staff marked as hard
(`borderline = yes` in their file), with the rule both of us applied:

| Post | Both said | The rule that decided it |
|---|---|---|
| sp_005 "Carbon plates aren't the whole story. My times dropped 2%…" | `analysis` | Rule 1. The opening clause reads like a hot take, but the two comparable 2% drops are the reason for the conclusion. |
| sp_010 "Three of us ran the same session… within 10 seconds" | `analysis` | Rule 1. A tiny, hedged sample, but the checkable observation is what the claim rests on. |
| sp_014 "Times are down about 3%… so obviously the shoes are the whole story" | `hot_take` | Rule 1's own example. The 3% decorates a conclusion already reached; "so obviously" does the work. |
| sp_019 "Nobody actually enjoys the long run…" | `hot_take` | Rule 2. It sounds reflective, but it's a general claim about everyone, not a report of the writer's run. |
| sp_020 "Every injury thread on this forum is the same injury…" | `hot_take` | Rule 1. Checkable in principle, but no count is given: asserted, not shown. |
| sp_028 "Third DNF this year…" | `reaction` | Rule 3 doesn't apply. "Third" is countable, but nothing is argued from it. |
| sp_029 "Just realised the shoes I set every PB in… wrong size" | `reaction` | Rule 2. A wry shoe observation, but it reports the writer's own realisation, now. |

**What the pattern in my disagreements tells me:** there's no disagreement
to find a pattern in, so the useful questions are whether the set was easy
and what a perfect score proves.

- **The set was easier than its hard cases suggest.** The posts are grouped
  by label in id order (sp_001–010 are all `analysis`, 011–020 `hot_take`,
  021–030 `reaction`), and most of each group is a clear example. My target
  of 21/30 was set for a set I expected to be messier. A 0.90 target (27/30)
  would have been a real test on this set.
- **It's good evidence on the boundary my model gets wrong *before* the
  fix, and weak evidence on the one it gets wrong *after*.** The rule-1 cases
  (sp_005, 010, 014, 020) are the same call as my `backed`/`bare_take` line,
  and I made all of them the staff's way. But the staff `reaction` is defined
  by *recency and the writer's own experience*, while mine is "no arguable
  claim once you strip the joke". The staff set has no sarcastic jabs about a
  company, which is where my `reaction`/`bare_take` line is hardest. So 30/30
  can't tell me whether I applied *my* `reaction` rule consistently across my
  230 r/Games posts. That's the question left open in What's Still Broken.

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:** `EPOCHS = 3` → `EPOCHS = 10` in notebook section 2. One
change only: same labels, same learning rate, batch size and max length, and
the same seeds 42, 7 and 2024. I picked 10 before running anything and didn't
try other values, so the after numbers aren't tuned against the test splits.

**Which diagnosis pointed at it:** criteria 1 and 2. The before model ended
training with a loss of 1.01–1.04, against ln 3 ≈ 1.10 for knowing nothing,
after only 30 optimizer steps, and it predicted `bare_take` for 34–35 of 35
posts. A model that hasn't fit its own training set needs more training
before relabeling or more data can show up in the numbers. Ten epochs is 100
steps. Training loss now ends at 0.69 / 0.72 / 0.73.

### Run Log — After

**Device:** `mps`, as before (`results_three_seeds_after.json`).

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.62 in every seed | 0.714 | 0.571 | 0.657 | MISSED |
| 2. Lowest per-label F1 | ≥ 0.45 in every seed | 0.667 (`backed`) | 0.000 (`reaction`) | 0.400 (`reaction`) | MISSED |
| 3. Trained − baseline accuracy | ≥ +0.10 (seed 42) | +0.371 (0.714 − 0.343) | +0.257 (0.571 − 0.314) | +0.343 (0.657 − 0.314) | MET |
| 4. Agreement with staff set | ≥ 21/30 (0.70) | 30/30 (1.00) | — unchanged by retraining | — | MET |
| 5. Most-sure − least-sure third accuracy | ≥ 0.15 in every seed | +0.091 (0.909 − 0.818) | +0.273 (0.818 − 0.545) | +0.273 (0.909 − 0.636) | MISSED |

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.714 | 0.571 | 0.657 | 0.143 |
| Macro F1 | 0.721 | 0.428 | 0.595 | 0.293 |
| F1 — `backed` | 0.667 | 0.632 | 0.667 | 0.035 |
| F1 — `bare_take` | 0.727 | 0.651 | 0.718 | 0.076 |
| F1 — `reaction` | 0.769 | 0.000 | 0.400 | 0.769 |

Confusion matrices after, all three seeds:

| Seed 42: true \ predicted | backed | bare_take | reaction |
|---|---|---|---|
| **backed** | 8 | 3 | 0 |
| **bare_take** | 4 | 12 | 0 |
| **reaction** | 1 | 2 | 5 |

| Seed 7: true \ predicted | backed | bare_take | reaction |
|---|---|---|---|
| **backed** | 6 | 5 | 0 |
| **bare_take** | 2 | 14 | 0 |
| **reaction** | 0 | 8 | 0 |

| Seed 2024: true \ predicted | backed | bare_take | reaction |
|---|---|---|---|
| **backed** | 7 | 4 | 0 |
| **bare_take** | 2 | 14 | 0 |
| **reaction** | 1 | 5 | 2 |

**Did it help, and how do I know:** Yes for what it targeted, and the
criteria verdicts barely moved. Before and after are on the same three splits:

| | Before (3 epochs) | After (10 epochs) |
|---|---|---|
| Accuracy, seeds 42 / 7 / 2024 | 0.486 / 0.486 / 0.457 | 0.714 / 0.571 / 0.657 |
| Macro F1, seeds 42 / 7 / 2024 | 0.269 / 0.269 / 0.209 | 0.721 / 0.428 / 0.595 |
| `backed` F1, mean of 3 | 0.111 | 0.655 |
| `reaction` F1, mean of 3 | 0.000 | 0.390 |
| Criteria MET / MISSED | 3 / 2 | 2 / 3 |

The model now learns the `backed`/`bare_take` boundary. `backed` F1 rose from
0.111 to 0.655 on average and is steady across seeds (spread 0.035). Its errors
go both ways (12 one way, 8 the other across seeds), which is what a genuinely hard pair looks like,
not a lopsided labeling problem. Two seeds now clear the 0.62 accuracy target,
but seed 7 (0.571) doesn't, so criterion 1 is still a miss.

What got worse: **criterion 5 went from MET to MISSED**, because seed 42's gap
is +0.091. That's not the model's confidence getting less useful: the
least-sure third went from 0.182 to 0.818 accuracy. With 11 posts per third,
one post is 0.091. When overall accuracy is 0.71, both thirds sit near the
ceiling, and there's no room for a 0.15 gap. I'm leaving the target where it
is. It measured the right thing, and I missed it on one seed.

The other new finding is stability. Accuracy now spreads 0.143 across seeds,
over the 10-point line, and `reaction` F1 ranges from 0.000 to 0.769. On 8
test posts per seed, `reaction` can't be measured stably. That's a real
diagnosis about the dataset's size, and it's the reason seed 7 fails both
criteria 1 and 2.



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->

After the improvement, three criteria are still missed (1, 2 and 5). Criteria
3 and 4 are met.

- **Criterion 2: `reaction` F1 0.000 / 0.400 on seeds 7 and 2024.** Across
  the three after-seeds, **15 of 24** real `reaction` posts were called
  `bare_take` and **0** `bare_take` posts were called `reaction`. That's
  lopsided, unlike the `backed`/`bare_take` pair. Two causes fit:
  `reaction` is my thinnest label (36 training posts, 52 total), and my own
  `bare_take` definition pulls jabs toward it ("sarcasm whose point is a clear
  verdict"). What I'd do next is add 30 more `reaction` posts, labeled under
  the same rule. Before that I'd re-read my 52 existing `reaction` labels
  against the "strip the joke, is an arguable claim left?" test, because
  rows 159–230 were never hand-checked. The agreement check can't settle
  which comes first: I matched staff 30/30, but their `reaction` rule is
  about recency, not my "no arguable claim" test, so it never tested my
  hardest `reaction` calls. I stopped because the unit allows one improvement, and the
  one the before-diagnosis pointed at was training.
- **Criterion 1: seed 7 accuracy 0.571.** Seed 7 gets 15 of 35 posts wrong,
  and 8 of those are its `reaction` posts, every one called `bare_take`. Two
  more right answers would clear 0.62, so this is mostly the same `reaction`
  problem, and the same fix applies.
- **Criterion 5: seed 42 gap +0.091.** I'd leave it. It's a ceiling effect on
  11-post thirds (above), and a bigger test split would show whether it's
  real. That needs more data overall, which is the same next step.

**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->

I meant `reaction` to mean "no arguable claim left once you strip the joke",
but the model learned something closer to "short and opinionated-sounding
means `bare_take`": 15 of 24 real `reaction` posts went to `bare_take`
across the after-seeds, and none went the other way. The `backed` line it did
learn, roughly the way I meant it, with errors in both directions on that
pair (12 `backed` called `bare_take` and 8 `bare_take` called `backed` across
three seeds) rather than a one-way slide.


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
