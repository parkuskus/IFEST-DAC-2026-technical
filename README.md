# IFEST DAC 2026

The qualifying round and the final used different datasets and asked different questions. The notebooks for both rounds are in this repository, but neither competition dataset was present in the source folder when this copy was made.

## Qualifying round

### The problem

Given a news headline and the article body, decide whether the headline matches the story. Label `1` means it does, while `0` means it does not. The metric is macro F1. About 90 percent of the training examples carry label `1`, so guessing the majority class performs poorly on the metric. Many of the difficult negatives share much of their vocabulary with the headline.

### Data

The official training file has 14,397 rows with `id`, `title`, `content`, and `label`. The test file has 3,603 rows without labels. A sample submission provides the `id,label` layout. The notebook expects these files under `data/`. It also refers to external Indonesian lexicons under `data/external_data/`, including place names, personal names, stopwords, and a thesaurus. The competition files and lexicons are absent from this copy. [`DATA.md`](DATA.md) records the source project's data inventory.

### What we built

[`penyisihan/notebooks/psi-1_notebook.ipynb`](penyisihan/notebooks/psi-1_notebook.ipynb) cleans the headline and body, then extracts lexical overlap and features for entities, numbers, negation, alignment, and retrieval. It trains a GRU cross-encoder from scratch. LightGBM, XGBoost, and CatBoost use those features and cross-encoder predictions. Each tree model runs with two seeds. An out-of-fold threshold chosen for macro F1 turns the averaged test scores into labels.

Pretrained model weights were prohibited in this round. The notebook trains its own models. The allowed uses of external data and the test set are described in the notebook and [`final/GUIDEBOOK DAC IFEST 2026.md`](final/GUIDEBOOK%20DAC%20IFEST%202026.md). The linked guidebook also covers the final round.

## Final round

### The problem

For each HUC12 watershed in the United States, estimate the risk of water stress in the following month. Each row supplies a 12-month history. Submissions contain a risk score between 0 and 1 for each `row_id`. Average Precision is the evaluation metric.

### Data

The official final-round dataset has about 378,780 training rows and 11,928 public test rows. It contains monthly river flows, water supply and withdrawals, climate medians, watershed attributes, and network identifiers. The notebook expects `train.csv`, `test_public.csv`, and `sample_submission.csv` in a Kaggle input directory. These files are not in this repository.

### What we built

[`final/notebooks/Final_DAC_2026_psi-1.ipynb`](final/notebooks/Final_DAC_2026_psi-1.ipynb) parses mixed number formats, detects inconsistent climate units, and builds within-row ratios to handle scale shifts in the public test set. Its four-member ensemble combines LightGBM with three RealMLP models trained on different ordinal targets. Each member is averaged across four seeds before the four predictions are averaged. The notebook writes `submission.csv` under its configured output directory. [Final-round details](final/README.md) cover the validation split and input requirements.

### What the final presentation adds

The team presented the work as D-RIVER. The notebook supplies the prediction part. The presentation then uses those scores to rank watersheds for inspection and studies the historical labels for longer-term planning. Those extra diagnostics are presentation analyses and are not reproduced by the archived final notebook.

On the presentation's top-100 inspection example, 93.0 percent of the highest-ranked watersheds had a positive label, compared with 24.6 percent for random inspection. The historical episode analysis estimates that 30.4 percent of observed stress episodes last at least three months. A stressed watershed remained stressed the following month 63.2 percent of the time in that analysis, while the transition rate from a non-stressed month to stress was 9.3 percent. These numbers describe the team's evaluation and historical data. They are not forecasts for an arbitrary new watershed.

The slides also compare stress frequency with population exposure. Some relatively low-population watersheds were stressed in around 40 to 46 percent of observed months, so population alone would miss them during screening. The team examined water withdrawals by sector and links between upstream and downstream watersheds as further context for inspection. Those comparisons do not establish that a particular withdrawal caused stress. Field checks and additional impact data would still be needed before acting on a ranking.

Neither notebook can be rerun from this archive alone. The qualifying notebook needs the missing CSVs and lexicons. The final notebook needs the missing CSVs and a valid `Settings.DATA_DIR`.
