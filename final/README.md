# IFEST DAC 2026 final round

## The problem

Predict whether a HUC12 watershed will experience water stress next month. The submission ranks test rows by risk, using a score from 0 to 1 in the `label` column. The competition evaluates Average Precision. Around 20 percent of the training rows have a positive label.

## Data

The official training set has 378,780 rows from 2,196 watersheds across 168 monthly origins. The public test set has 11,928 rows from 2,982 watersheds across four origins. Each row carries a 12-month window of baseflow, quickflow, water supply, and withdrawals. Climate medians, watershed area, population estimates, and downstream identifiers provide additional context. `row_id` identifies test rows and must be preserved in the submission.

The notebook expects `train.csv`, `test_public.csv`, and `sample_submission.csv` under the directory set by `Settings.DATA_DIR`. None of those files were present in the source folder. The default setting points to a Kaggle input path and must be changed for a local run.

## The notebook

[`notebooks/Final_DAC_2026_psi-1.ipynb`](notebooks/Final_DAC_2026_psi-1.ipynb) reads the raw files as strings because the numbers mix decimal and thousands separators. It repairs climate values with inconsistent units and builds 139 ratio features. Ratios were chosen because the public test set changes the scale of raw flow values for individual watersheds.

The model blends four members. One is LightGBM. The other three are RealMLP models that predict different ordinal bins for the next month's supply ratio. Four seeds are used per member, followed by an equal-weight average across members. Validation uses held-out time blocks with an 11-month gap between training and validation windows. This is closer to the test setting than a random row split.

The output goes to `Settings.OUTPUT_DIR/submission.csv` with `row_id,label` columns. The notebook checks that each test row appears once and that the scores are finite. A full run requires the competition data, LightGBM, PyTorch, pytabkit, and the other imported Python packages.

## Why the pipeline is shaped this way

The training and public test files mix number formats, and some climate values use the wrong unit. A parser that treats every comma as a thousands separator would quietly corrupt those rows. The notebook repairs the values before it derives features. Its 139 ratios preserve relationships between supply, withdrawals, and climate even when the raw measurements for a test watershed have been multiplied by a different scale factor.

The presentation compares LightGBM with the ordinal RealMLP variants and reports that the combined prediction performs better than the individual candidates in its evaluation. It also identifies current supply and the next month's climate median as major drivers of the model score. Lower supply tends to raise predicted risk. This is a feature attribution result, not proof that one measured variable alone causes the outcome. The broader risk triage and persistence findings are summarized in the [project README](../README.md).

The final-round rules forbid external datasets and pretrained model weights. See [`GUIDEBOOK DAC IFEST 2026.md`](GUIDEBOOK%20DAC%20IFEST%202026.md) for the original summary kept in this archive.
