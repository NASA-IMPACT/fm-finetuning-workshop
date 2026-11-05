# Example: map flood water with Prithvi

A worked example for the FM Finetuning Studio hands-on. You play a scientist on a disaster
response team who wants a map of flooded areas after a storm. The studio's agents interview
you, check the data, pick a Prithvi model setup, train it on a shared GPU and write up what
happened. You only answer questions.

| | |
|---|---|
| Dataset | [Sen1Floods11](https://huggingface.co/datasets/KozaMateusz/sen1floods11): Sentinel-2 satellite images of 11 flood events with hand-drawn water masks |
| Model | Prithvi-EO-2.0 (300M), NASA and IBM's Earth-observation foundation model |
| Time | about 60 minutes in full, about 25 if you skip the data audit |
| You need | your studio login, and nothing else: no code, no data on your laptop |

> The agents write their questions fresh each time, so yours will not be word for word what
> is shown here. Each step below says what the question is *about* and what to answer.
> Short, plain answers work best: type the way you would talk to a colleague.

---

## How a session flows

```
 Stage 0   Your goal ............ you describe the problem, the data is fetched
 Stage 1   Requirements ......... a short interview from the other side's point of view
 Stage 2   Data audit ........... checks and plots on the actual files  (~35 min)
 Stage 3   Model shortlist ...... ranked ways to train a model; you pick
 Stage 4   Experiment plan ...... a verified plan; you say "go"
 Stage 5   Training ............. a quick test run, then training on the GPU
 Stage 6   Final report ......... the whole story in one document
```

At the end of every stage the studio stops and asks **"Stage N is done … anything to add?"**
with a **go on** button. Nothing moves on, and nothing trains, without you clicking it.

Every stage writes pages for you, as Markdown and PDF, in the **Files** panel on the right
(`stage0_input/`, `stage1_requirements/` … `stage6_report/`). Open them as they appear.

---

## Stage 0: tell it the problem

Start a new session and paste:

```
I want to map which areas are flooded after a storm from satellite images, for a disaster response team. Data: https://huggingface.co/datasets/KozaMateusz/sen1floods11
```

It fetches a **sample** of the dataset (500 of its 893 files; plenty for a 2-epoch run) and
writes a quick look at it. Then it asks about a model and how you will judge the result:

```
Prithvi EO, you pick the rest. Just mark each pixel water or not. It's good if it matches what a person would draw by hand. Missing water is worse than a false alarm.
```

If it asks about constraints or deadlines:

```
No, nothing hard. Just keep it quick.
```

**Stage 0 is done** → click **go on**.

Look at: `stage0_input/goal.md` (your goal in your words) and `stage0_input/dataset_sen1floods11.md`.

---

## Stage 1: the requirements interview

The **Requirements Analyst** interviews you from the side you are *not* on. You are the
flood scientist, so it brings the machine-learning expertise: before each question it shows
a recommendation with the papers behind it, then asks one thing only you can decide.

| It asks about | Answer | Why this answer |
|---|---|---|
| Your background | `1` (flood, water or disaster-response science) | You are playing the scientist |
| All water, or only new flood water? | `1` (all visible water, lakes and rivers too) | The dataset's hand labels mark all water; "new flood only" would need a before-the-storm image this data does not have |
| Clouds or gaps hiding part of the area | `1` (use as an aid, mark hidden areas) | A response map must say where it cannot see, never call it dry |
| Uncertain water edges | `1` (count them as possible water) | Follows from "missing water is worse" |
| Which regions or flood types matter | `all of them, it's a first test` | Fine for a first run |
| Anything else | your own words, or `you choose` | "you choose" lets the agent decide in its role |

Two things worth reading while it runs: the **recommendation** above each question (it
cites real papers you can open) and `stage1_requirements/requirements.md` at the end:
what must and must never happen, and every word from the ML side explained in plain terms.

---

## Stage 2: the data audit

It asks **"Shall I look at the data now?"** → `1` (yes).

**Before the checks** it asks up to three things the files cannot tell it, each with its
own understanding. For this dataset these are right; click **yes, that's right** (`1`):

- the mask codes: **1 = water, 0 = not water, −1 = ignored**
- the 13 image bands: Sentinel-2's **B1 to B12 in order, with B8A after B8**
- the image values: **reflectance stored × 10,000**, so a scale of 0.0001

Then it shows the **audit plan** (5 checks) and later the **exploration plan** (4 ways
to look at the data). Click **go** on each.

Each check reports in the chat as it finishes, with its page in `stage2_data/checks/`; each
exploration comes with plots in `stage2_data/exploration/`. This is the long stage
(about 35 minutes), so read them as they arrive. Things to notice in this dataset:

- **The split.** Sen1Floods11 ships its own training, validation and test lists: 140, 48
  and 47 images of the sample. The **15 Bolivia images** are in none of them on purpose;
  the dataset keeps Bolivia as a separate test of unseen floods.
- **Water is rare**: about 1% of a typical image. That is why missed water needs extra
  weight in training.
- **Near-infrared light separates water best** (band 8). Water is dark there.
- **Pakistan is the hard case**: muddy flood water looks less different from land.

**Stage 2 is done** → click **go on**. The data report is `stage2_data/report.md`;
every script that ran, with its output, is in `stage2_data/audit_outputs.zip`.

> **Short on time?** When it offers the audit, answer
> `skip the audit and EDA, go to the shortlist` instead. You lose the checks and plots,
> not the training.

---

## Stage 3: pick a model setup

It lists about four candidates, each with its sources, and asks which to plan. Click the
one that **fully fine-tunes Prithvi with a UperNet decoder** (here it was number 3,
`full-prithvi-upernet`), then **Plan the selected**. Or type its number.

Why that one: it follows NASA's official Prithvi Sen1Floods11 example, and with training
capped at 2 epochs a fully fine-tuned model learns far more than a frozen one. Pick one
for this session: candidates run one after another, not in parallel.

Look at: `stage3_models/shortlist.md`, with what each backbone was trained on and where
every choice comes from.

---

## Stage 4: the experiment plan

The FM Scientist writes the plan, the Data Scientist checks it against your data, and a
Verifier reviews it. This takes about 10 minutes. Then:

**"Experiment plan 1 is verified and written for you … Shall I build and run it?"** → **go**

Look at: `stage4_plans/experiment_1.md`: what will run, why this model and this way of
training, the settings, what the data check and the verifier said, and the risks.

---

## Stage 5: training

Nothing to type. The Code Adapter writes the training setup from NASA's official example,
runs a quick test on a few images, then starts the real run. A progress bar shows the epochs.

- Training is **capped at 2 epochs for everyone** so the shared GPUs stay free. It takes a
  few minutes.
- When it finishes, the result is measured on the held-out test images (never seen in
  training), judged against the plan, and explained.

If it says the result is ready, answer `go on`. At **"Stage 5 is done"** → **go on**.

Look at: `stage5_experiments/experiment_1.md` (the numbers, the training curves, what
stood out, what to try next) and `experiment_1_outputs.zip` (config, code, logs).

**What to expect.** In our test run of exactly this example:

| Measure | Result after 2 epochs | Published, full training |
|---|---|---|
| Water recall (share of real water found) | about 56% | |
| Water IoU (overlap with the hand-drawn water) | about 28% | 82.6% |

The model clearly learns and leans towards finding water, as you asked. It is far from the
published number because it trained for 2 epochs on 140 images. That is the point of the
session: the whole path, not a finished model.

---

## Stage 6: the final report

`stage6_report/final_report.md` (and `.pdf`): the problem, what the interview settled, the
dataset and its split, what the audit found with its plots, the literature, the models
considered, the plan and why, the run and its numbers, the limits and what to do next.
It is written to hand to someone who was not there. **Download all** in the Files panel
gives every stage as one zip.

---

## Useful things to type, any time

| Type | What happens |
|---|---|
| `explain` | the full picture of where you are |
| a question, e.g. `why did check 3 not pass?` | it answers from the pages before moving on |
| `skip the audit and EDA, go to the shortlist` | jumps past stage 2 |
| `run experiment 1 again, same plan` | another training run of the approved plan |

You can always answer in your own words instead of clicking a number.

---

## Try it your way

Once the example has run, change one answer and see what changes:

- **"Only land newly flooded by the storm"** in stage 1: watch the interview and the audit
  explain why this data cannot support it.
- **Pick the frozen probe** in stage 3 instead: compare its numbers with the full fine-tune.
- **Your own data**: paste a public link (Hugging Face, Zenodo, S3) or attach files up to
  512 MB, and describe your problem the same way.

## A faster second example: burn scars

Same flow, different problem. Start a new session and paste:

```
I want to map burned areas after a wildfire from satellite images, for a forest service that plans recovery. Data: https://huggingface.co/datasets/ibm-nasa-geospatial/hls_burn_scars
```

```
Each pixel burned or not burned. It works if it matches the burn scars the agency mapped by hand.
```

```
Prithvi EO. You suggest the rest.
```

In the interview, missing a real burn is worse (`1`), and use it for new fires after a
benchmark (`3`). Before the checks: masks are 1 = burned, 0 = not burned, −1 = no data. The
dataset keeps its own `training/` and `validation/` folders, and the validation folder is the
held-out test.

---

## If something goes wrong

- **The chat stops responding or the connection drops:** reload the page. Your session and
  its files are kept; open it again from **Sessions**.
- **A question you do not understand:** answer `I don't know` or `you choose`. Both are
  valid; the agent decides in its role and writes down that it did.
- **A step fails:** the chat says what failed and what it will do. Answer `go on`, or ask
  `what went wrong?`.
- **Anything else:** raise your hand; an organiser will come over.
