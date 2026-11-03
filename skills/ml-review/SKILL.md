---
name: ml-review
description: Review data processing and machine learning pipeline code (pandas, scikit-learn, notebooks, SageMaker) for data leakage, silent data bugs, evaluation mistakes and reproducibility issues. Use when writing or reviewing training, feature engineering or data transformation code.
argument-hint: [file, folder or notebook]
---

# ML pipeline review: $ARGUMENTS

Data and ML bugs rarely crash; they silently produce wrong numbers. Look for those first.

## Data leakage
- Scalers, encoders, imputers or feature selection fitted before the train/test split, or outside a `Pipeline` in cross-validation.
- Target leakage: features computed from the target, or using information only known after the prediction moment (e.g. final reimbursement amount, processing end date).
- Random splits on temporal data where a time-based split is needed; the same entity (member, claim, provider) in both train and test when it should be grouped.

## Silent data bugs (pandas)
- `merge`/`join` that duplicates or drops rows: check row counts before and after, use `validate="one_to_one"` / `"many_to_one"`.
- Implicit dtype changes: IDs read as floats, leading zeros lost, dates parsed with the wrong day/month order or timezone.
- NaN handling: `fillna(0)` where 0 is meaningful, NaN silently dropped by `groupby` or comparisons.
- Chained assignment / `SettingWithCopyWarning`, in-place mutation of a DataFrame used elsewhere.
- Filters and joins on string columns with inconsistent case or whitespace.

## Evaluation
- Metric suited to the problem (imbalanced classes: precision/recall, PR-AUC; regression with outliers: MAE vs RMSE) and compared against a simple baseline.
- No tuning on the test set; threshold choice made on validation data.
- Results reported with enough context (class balance, sample sizes, variance across folds).

## Reproducibility
- Seeds set for splits and models; dependency versions pinned.
- Input data version or snapshot date recorded with the model.
- Paths, dates and parameters in config, not hardcoded in notebook cells.
- Notebook logic that matters for production lives in importable modules with tests.

## Performance
- `iterrows`/`apply` row loops where a vectorized operation works.
- Loading full tables when only some columns or partitions are needed.

## Personal data
- Personal or health data in notebook outputs, logs, plots or files committed to git.

## Report

For each verified issue: `file:line` (or notebook cell), what goes wrong and its effect on the results, and the fix. Order by impact on correctness of results first. Don't edit unless asked.
