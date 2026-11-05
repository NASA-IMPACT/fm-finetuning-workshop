# Example: map wildfire burn scars

You fine-tune Prithvi-EO-2.0 to mark which pixels burned in HLS satellite images. The data
is already on the studio's machine, so there is nothing to download or upload.

## The data

HLS Burn Scars (NASA and IBM), in a folder every participant can read:

```
/opt/fm/data/shared/hls_burn_scars/training
```

| | |
|---|---|
| Images | 540 chips, `*_merged.tif`: 512 x 512 pixels, 30 m, 6 bands (blue, green, red, NIR, SWIR 1, SWIR 2) |
| Labels | 540 masks, `*.mask.tif`: `1` burned, `0` not burned, `-1` no data |
| Also there | `../validation/`: 264 more chips and masks |

Only about 11% of the pixels are burned. A model that answers "not burned" everywhere is 89%
accurate and useless, so the number that matters is the **IoU of the burned class**: the
overlap between the predicted and the true burned area, divided by their union (1 is
perfect, 0 is no overlap).

## How to read this page

Each step shows what the chat asks (shortened) and what to type. When the chat shows
numbered options, click one or type its number. Questions marked *(asked by code)* appear
word for word; the others are written by the agents and change from run to run.

## 1. Start

Sign in, then click **Start a new session**.

## 2. The problem

> What do you want to solve? Tell me the problem in your own words: what you would like a
> model to do, and for whom. *(asked by code)*

Type:

```
Map wildfire burn scars from HLS satellite chips, which pixels burned, for a fire-monitoring team that draws burn perimeters.
```

## 3. Data and model

The chat asks whether you have data, where it is and in what format, and whether you have a
model in mind.

Type:

```
Yes, the data is at /opt/fm/data/shared/hls_burn_scars/training: 540 six-band HLS GeoTIFF chips (*_merged.tif) with burn masks (*.mask.tif: 1 burned, 0 not burned, -1 no data). Use Prithvi-EO-2.0-300M-TL with a frozen backbone and a UNet decoder. Success is burned-class IoU on held-out chips, with precision and recall alongside. One experiment only.
```

- **Frozen backbone**: the pretrained Prithvi weights stay fixed and only the small decoder
  learns. It trains fast and fits a shared GPU.
- The agents check every claim against the files, so a wrong one is caught. A clear one
  saves questions.

## 4. Stage 0 is done

> Stage 0 is done. Written for you, each as Markdown and PDF: ... Anything to add before we
> go on? *(asked by code)*

Click **go on**. The goal and the data description are in **Artifacts**, under
`stage0_input/`.

## 5. The requirements interview

The first question is always which side you are on:

> Before anything else: are you on the science side of this, the machine-learning side, or
> both?

Pick yours. The agents then ask from the other side, one question at a time, often with a
recommendation and its sources first. For example:

> Should the model-selection rule prioritize burn recall or overall agreement with the
> outlines?

Answer with a number, in your own words, or **you choose**. For a fire-monitoring team, a
missed burn usually costs more than a false alarm, so **few missed burned areas** is a
reasonable answer.

## 6. Permission to read the data

> Shall I look at the data now? I would read it and run some checks on it; nothing in your
> folder is modified. *(asked by code)*

Click **yes**. It may first ask about something the files cannot show:

> Before the checks, 2 things the data cannot tell me and no source states (1 of 2): ...
> My understanding: ... *(asked by code)*

Click **yes, that's right**, type the right answer (for example,
`-1 means no data, not a class`), or say you don't know.

## 7. Data checks and exploration

> Before anything is planned on your data, I want to run 4 checks on it. ... Shall I run
> these? *(asked by code)*

Click **go**, or ask for one more, for example
`add a check that every chip has a mask of the same size`. Each check reports when it
finishes:

> **Check 1 of 4 done: inventory and access**: 2 of 2 met.

Then the same for the exploration (band distributions, correlations, how burned pixels
relate to the bands), with plots. Click **go** again. This step takes a few minutes; every
check has its own page under `stage2_data/`.

## 8. Stage 2 is done

> Stage 2 is done. ... Anything to add before we go on: a question about a finding, one of
> the decisions the report leaves to you, more data? *(asked by code)*

Open `stage2_data/report.md` in **Artifacts**. Its **Usable for the task** line gives the
verdict (yes, with conditions, no, or unknown), and the rows under it say why. Ask about anything that surprises
you, for example `how many chips have no burned pixels at all?`, then click **go on**.

## 9. Pick a model

> **Which experiments should I plan?** *(asked by code)*
>
> 1. semanticsegmentation-prithvi_eo_v2_300_tl-unetdecoder-freeze_backbone (recommended)
> 2. ...

Each name reads as *task - backbone - decoder - how it is trained*. Details and sources are
in `stage3_models/shortlist.md`. Pick the one that matches step 3 (`prithvi_eo_v2_300_tl`,
`unetdecoder`, `freeze_backbone`), usually the recommended one, then click the confirm
button. Pick one only: every pick is a training run on the shared GPU.

## 10. The plan

The FM scientist writes the plan, the data scientist checks it against the data, and the
verifier checks it against the sources. This takes a few minutes. Then:

> Experiment plan 1 is verified and written for you: stage4_plans/experiment_1.md (and
> .pdf). Shall I build and run it? *(asked by code)*

Read the plan, then click **go**. To change something, say so, for example
`use UperNetDecoder instead`: it re-plans and asks again.

## 11. The run

The code adapter writes the training code, a quick test checks it is wired correctly, and
training starts on the GPU. A progress line shows where it is:

> Experiment 1, attempt 1 · epoch 1 of 2 · ... elapsed

Runs on this machine stop after 2 epochs (passes over the training chips), whatever the plan
says, so the shared GPU has room for everyone. The plan keeps its own number. Ask `where has it reached?` at any time. When it ends:

> **Experiment 1 finished** (attempt 1, ...). The job reports ... These are its own numbers;
> the evaluator measures the predictions next. Say "go on" and the result is measured,
> judged and explained. *(asked by code)*

Click **go on**. The evaluator scores the model on held-out chips it never saw during
training. It recomputes the numbers itself rather than trusting the training job's own.

## 12. Stage 5 is done

> Stage 5 is done. ... Anything to add before we go on: a question about an experiment,
> another run you want, a change? *(asked by code)*

The experiment page, with its scores and training curves, is under `stage5_experiments/`.
Click **go on**.

## 13. The final report

The report generator writes `stage6_report/final_report.md` (and .pdf): what was tried, what
was measured, what it means and what to do next. Open **Artifacts** and click
**Download all** to keep everything.

## Reading the score

- **Burned-class IoU**: the main number. The "not burned everywhere" model scores 0.
- **Precision**: of the pixels marked burned, the share that really burned. Low precision
  means perimeters drawn too large.
- **Recall**: of the pixels that really burned, the share the model found. Low recall means
  missed burns.
- A short run on a shared machine gives a starting point, not a tuned model. The report
  says what to change next.

## Try next

- Answer the interview from the other side (science or machine learning) and see how the
  questions change.
- At step 10, ask for `UperNetDecoder` instead of the UNet decoder.
- Run the same steps on your own data, with a public link (see
  [Bringing your data](../README.md#4-bringing-your-data)).
