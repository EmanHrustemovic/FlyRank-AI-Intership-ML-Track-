# Content Decline Scoring — FlyRank ML Internship Capstone

**Author:** Eman Hrustemović
**Track:** FlyRank AI Internship — Machine Learning
**Mentor:** Mirza Ašćerić

## What it does, and for whom

This project flags which pages on a website are losing search ranking, so an SEO
team doesn't have to manually review thousands of pages by hand every month.

Given a page's monthly search performance (clicks, impressions, average position,
CTR, and how those numbers moved), the model outputs a **decline score**: how
likely it is that a page needs attention. The output is a ranked queue — the
pages most worth reviewing first are at the top.

**Who this is for:** SEO analysts or content teams who currently triage page
performance manually, and want a first-pass filter instead of eyeballing a
spreadsheet of thousands of rows.

## Setup (should work for a stranger, start to finish)

1. Clone the repo:
   ```bash
   git clone https://github.com/EmanHrustemovic/FlyRank-AI-Intership-ML-Track-.git
   cd FlyRank-AI-Intership-ML-Track-
   ```
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. The anonymized sample dataset ships with the repo at `data/raw/` — no
   credentials or data access request needed to run the base pipeline.
4. Run the capstone pipeline:
   ```bash
   python work/<your_capstone_folder>/run_capstone.py
   ```
   > ⚠️ **Eman — confirm this path against your actual `work/` folder before
   > publishing.** Replace `<your_capstone_folder>` and the script name with
   > whatever you actually committed.
5. Outputs (ranked queue, charts, metrics) are written to `outputs/` (or your
   capstone's own output folder — confirm the exact path).

## Usage example

```bash
python work/<your_capstone_folder>/run_capstone.py --input data/raw/content_refresh_anonymized.csv
```

This produces a CSV of pages ranked by decline score, highest-priority first —
open it directly, or load it into a spreadsheet to review the top N pages.

## Architecture (high level)

```
Raw page-month data (clicks, impressions, position, CTR)
        │
        ▼
Feature prep — one row = one page-month
        │
        ▼
Baseline rule: flag if CTR is far below what position predicts
        │
        ▼
Signal test: content staleness (discarded — no signal)
             CTR-vs-position gap (kept — held up under testing)
        │
        ▼
Logistic Regression model, trained on the surviving signal
        │
        ▼
Evaluation under a CLIENT-GROUPED split (not random split —
this matters, see Limitations)
        │
        ▼
Ranked output: pages sorted by decline score
```

## Eval results (v2)

| Metric | Baseline (hand rule) | Model (Logistic Regression) |
|---|---|---|
| Precision@50 | 0.70 | **0.96** |

- Baseline: flag a page if its CTR is far below what its search position would predict.
- Model: trained on top of that, then re-evaluated under a stricter **client-grouped
  split** — pages from the same client never appear in both train and test, so the
  model can't just memorize client-specific quirks.
- The model's improvement held up under that stricter split, which is why it's
  trusted over the baseline.

## Limitations (named, not hidden)

- **Client-grouped split reduces effective training data.** Since whole clients
  are held out for testing, the model sees less variety per training run than a
  random split would give it — this is a deliberate honesty trade-off, not an
  oversight.
- **Content staleness was tested and discarded as a signal** — it looked
  plausible going in, but didn't hold up. Including it would have made the
  model look more sophisticated without making it more accurate.
- **This is decision-support, not a guarantee.** A high decline score means
  "review this page first," not "this page is definitely declining for X
  reason." Framing it as anything stronger would overstate what the model
  actually knows.
- **Trained on an anonymized sample slice**, not the full ~79M-row warehouse —
  results on the full dataset may shift.

## Built with AI — what and how

I used Claude to help draft this README and the double-submit / accessibility /
SEO fixes on my portfolio site, and as a thinking partner for structuring the
capstone pipeline and reasoning through which evaluation split to trust. The
model choice, the decision to test and discard content staleness as a signal,
the client-grouped split, and the final Precision@50 numbers were run and
checked by me against the actual data — Claude didn't generate the results,
it helped me think through how to validate them honestly.

## Demo

[Link to 3–5 minute demo video — add after recording]
