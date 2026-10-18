# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

Every criterion below is measured on the held-out **test split** that the
notebook makes (15% of `labels.csv`, so about 30 posts). Where it says "all
three seeds", it means the three runs in notebook section 6 (seeds 42, 7 and
2024), and the target has to hold in **each** of them, not on average.

## 1. Overall accuracy is at least 0.62 in all three seeds

`accuracy` in section 6's results is at least **0.62** for seed 42, seed 7
**and** seed 2024. Compare the unrounded value stored in the results, not a
rounded display.

**Why this target:** Before labeling, I expect `bare_take` to be about half of
my r/Games comments. In my 40-comment reading sample, verdicts with nothing
behind them were the most common kind. So a model that always guesses
`bare_take` would score about 0.50, and three-way chance is 0.33. 0.62 means
being right on roughly 4 more test posts out of 30 than that lazy guess, in
every seed. That's my line for "it learned something". I didn't go higher
because the `backed`/`bare_take` boundary turns on whether a fact is used *as
evidence*. That's a judgement my decision rule needed three exceptions to pin
down, and DistilBERT will probably lean on surface cues like digits and `$`
signs, which also appear in posts where the number is just the topic ("Paying
$70 for this is insane").

---

## 2. No label's F1 drops below 0.45 in any seed

For every one of `backed`, `bare_take` and `reaction`, the per-label F1 is at
least **0.45** in all three seeds. That's 9 numbers, and all 9 must clear it.
A label the model never predicts gets an F1 of 0 (that's what the notebook
reports, with `zero_division=0`), and that counts as a fail.

**Why this target:** The overall number can hide one label the model never
gets right. The stress test suggested `reaction` will be my thinnest label:
none of the AI's 8 boundary posts landed there. If it's around 20% of 200
posts, only about 6 reach the test split, and one wrong guess moves its F1
by roughly 0.1. 0.45 is deliberately looser than criterion 1 for that reason.
It still rules out a label the model gets right less than half the time.

---

## 3. The trained model beats the zero-shot baseline by at least 0.10 accuracy

Run `python baseline.py` on the committed `test_split.csv`. The trained model's
seed-42 accuracy (from `results.json`) is at least **0.10 higher** than the
baseline's accuracy on those same posts, the `accuracy` line that
`baseline.py` prints. The baseline is a zero-shot classification pipeline: it
always picks one of my three labels and gives the same answer every run, so
there are no unscorable outputs and nothing to re-run.

**Why this target:** The baseline gets my exact definitions from
`label_definitions.txt`, and `reaction` vs the other two is easy to describe
in words ("a joke or a feeling, no claim"). So a general model might do well on
that part with no training at all. Where I expect fine-tuning to earn its keep
is the `backed`/`bare_take` line, where my rules ("a number that's the topic
doesn't count", "naming a game isn't stating a fact") are r/Games-specific and
not obvious from a one-line definition. 0.10 is about 3 posts out of 30: a
small edge, but more than one lucky post. If training doesn't buy at least
that, it isn't worth the extra step.

---

## 4. I agree with the staff labels on at least 21 of the 30 staff posts

Label the 30 posts in `data/staff_posts.csv` under the staff definitions in
`data/staff_taxonomy.md`, then run `python agreement.py`. The agreement rate
it prints is at least **0.70** (21 of 30). These posts are labeled with the
staff's own label names (`analysis` / `hot_take` / `reaction`), so no mapping
to my labels is involved. The labeling is my judgement, but the rate is
arithmetic anyone can re-run.

**Why this target:** This is the only check on whether labels like mine are
applied consistently. The staff taxonomy (`analysis` / `hot_take` /
`reaction`) lines up closely with mine (`backed` / `bare_take` / `reaction`),
and their `analysis` rule also says the fact "has to do work in the argument",
which is my exact hardest boundary. So I'd expect to agree on most posts. I
didn't set it higher, because their `reaction` is tied to "something that just
happened", while mine is about having no arguable claim. Posts that react to
news *with* a verdict will split between those two readings. 21 of 30 allows
up to 9 of those disagreements.

---

## 5. Confident predictions are right more often than unsure ones by at least 0.15

In each seed's `confidence` report, `accuracy_most_confident_third` minus
`accuracy_least_confident_third` is at least **0.15** in all three seeds.
"Thirds" are cut the way the notebook's `confidence_report` does it: test posts
are ranked by the model's top softmax probability, and the lowest and highest
`len(test) // 3` posts are compared. With 30 test posts that's 10 each.

**Why this target:** If the model's confidence means anything, a user could
trust its sure calls and double-check its unsure ones. I expect the clear cases
to be short jokes (`reaction`) and posts with an obvious stat or quote
(`backed`). The unsure ones should be the posts my decision rule had to
handle: games named without a fact, numbers that are the topic, sarcastic
verdicts. Each third is about 10 posts, so 0.15 is a gap of 1.5 posts. I
didn't go higher because with 10 posts per third, one swapped post moves
either number by 0.10.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
