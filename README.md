# Automated Spam & Fraud Message Detection

Deep Learning course project proposal — supporting code and data exploration.
**Author:** Haidar (KFUPM, Software Engineering — AI concentration)

The full written proposal is in `proposal/` as both an editable Word file
([`Haidar_DL_Project_Proposal.docx`](proposal/Haidar_DL_Project_Proposal.docx))
and a submission-ready PDF export
([`Haidar_DL_Project_Proposal.pdf`](proposal/Haidar_DL_Project_Proposal.pdf)).

## Problem

Given the raw text of a single inbound message, predict whether it is
**spam/fraud** or **legitimate ("ham")**. This is a **binary text
classification** task (NLP).

**Motivation:** Many wholesale/logistics businesses now take orders and
inquiries over messaging channels (SMS/WhatsApp) instead of only through a
web form. These channels attract the same premium-rate-number scams and
phishing spam that plague personal SMS. An automatic filter that flags
likely spam/fraud before it reaches a
human agent reduces wasted staff time and scam risk.

## Dataset

[SMS Spam Collection](https://archive.ics.uci.edu/dataset/228/sms+spam+collection)
(Almeida & Hidalgo, 2011) — CC BY 4.0 license, hosted on the UCI ML Repository.
Also mirrored on [Kaggle](https://www.kaggle.com/datasets/uciml/sms-spam-collection-dataset)
(same corpus, uploaded by `uciml`). Bundled directly in this repo as
`data/sms.tsv` (478 KB — small enough to commit), so no download is needed.

| | |
|---|---|
| Total messages | 5,574 real, individually labeled SMS messages |
| Labels | Binary: ham (4,827, 86.6%) / spam (747, 13.4%) — imbalanced |
| Message length | Ham: mean 14.3 words; Spam: mean 23.9 words |
| Duplicates | ~403 exact duplicate messages in the raw corpus (known property) |

## Repository layout

```
.
├── proposal/
│   ├── Haidar_DL_Project_Proposal.docx  # the 2-page written proposal (editable)
│   └── Haidar_DL_Project_Proposal.pdf   # same content, exported to PDF for submission
├── data/
│   └── sms.tsv                          # the full dataset (bundled, 478 KB)
├── figures/
│   ├── sample_messages.png              # Figure 1: representative samples
│   └── length_distribution.png          # Figure 2: length distribution by class
├── dataset_stats.json                   # summary stats (counts, lengths, duplicates)
├── src/
│   ├── make_figures.py                  # reproduces both figures + dataset_stats.json
│   └── build_split.py                   # builds the stratified train/val/test split
├── splits/                              # output of build_split.py (train.csv, val.csv, test.csv)
├── requirements.txt
└── README.md
```

## Reproducing the figures

```bash
pip install -r requirements.txt
python src/make_figures.py
```

This regenerates `figures/sample_messages.png`, `figures/length_distribution.png`,
and `dataset_stats.json` directly from `data/sms.tsv` — no download needed.

## Train / validation / test split

Exact duplicate messages (~403 of 5,574) are removed first, leaving 5,171
unique messages (12.6% spam), so the same message never appears in two
different subsets. The deduplicated corpus is then split **70% / 15% / 15%
per class** (stratified, fixed seed), preserving that ratio in every subset:

- Train: 3,620 messages (70%)
- Validation: 776 messages (15%) — tuning, model selection, threshold calibration
- Test: 775 messages (15%) — held out untouched, used once for final evaluation

```bash
python src/build_split.py --data data/sms.tsv --out-dir splits
```

This writes `splits/train.csv`, `splits/val.csv`, and `splits/test.csv`, each
a `label,message` table.

## Validation metric

**F1-score on the spam (positive) class**, computed on the validation split.
With only 13.4% of messages labeled spam, plain accuracy is misleading — a
model predicting "ham" for everything already scores 86.6% accuracy while
catching zero spam. F1 balances precision (not misflagging real customer
messages) against recall (actually catching spam/fraud), the right trade-off
for a business messaging filter. Precision and recall are also reported
individually to make that trade-off visible when tuning the decision
threshold.

## A data-quality note (worth knowing before you train)

`data/sms.tsv` is tab-separated, but a couple of messages contain literal
double-quote characters that confuse Python's default `csv.reader` quoting
and silently drop 2 rows (5,572 vs. the official 5,574). Both scripts here
parse with a plain `line.split("\t", 1)` instead to avoid that. Keep this in
mind if you swap in `pandas.read_csv` or another CSV loader — set
`quoting=csv.QUOTE_NONE` or split manually.

## Citation

Almeida, T. A., Gómez Hidalgo, J. M., & Yamakami, A. (2011). "Contributions
to the study of SMS spam filtering: new collection and results."
*Proceedings of the 11th ACM Symposium on Document Engineering*.
