# Python review checklist

## Correctness
- Mutable default arguments, late-binding closures in loops, generators consumed twice.
- Timezone-aware datetimes used consistently; explicit date formats when parsing.
- Broad `except Exception` or bare `except` hiding real errors; exceptions re-raised with context (`raise ... from e`).
- Files opened with `encoding="utf-8"` and with context managers.
- Return types consistent (no function returning a list sometimes and `None` other times without reason).

## Data (pandas)
- Row counts checked around `merge` / `join` (`validate=` where the relationship is known).
- Explicit `dtype=` for IDs and codes when reading CSV or Excel.
- NaN handling deliberate (`groupby(dropna=...)`, `fillna` values that are not real data).
- No chained assignment; copies made explicitly when a DataFrame is modified.
- Vectorized operations instead of `iterrows` / `apply` on large frames.

## ML, ASR and LLM code
- Preprocessing identical at training and inference.
- No fitting on test data, no information from the future in features.
- Model and library versions pinned together; seeds set where results must be reproducible.
- Model output parsed defensively and validated; prompts and templates versioned.
- Evaluation compared against a baseline, with the metric suited to the problem.

## AWS and infrastructure
- S3 listing paginated; keys built without double slashes; S3 event keys URL-decoded.
- Lambda handlers idempotent, clients created outside the handler, nothing written outside `/tmp`, timeouts handled.
- No hardcoded credentials, bucket names or account ids that should be configuration.
- IAM permissions as narrow as the code needs.

## Personal data
- No personal or health data in logs, exceptions, notebook outputs, fixtures or prompts sent to external services.

## Structure and standards
- Imports at the top; helpers after the function that calls them, in call order.
- Logic that production depends on lives in importable, tested modules, not only in notebooks or Streamlit pages.
- Configuration (paths, dates, model names, thresholds) not hardcoded in the middle of code.
- Type hints where the codebase uses them; consistent with the existing style (ruff, formatter).

## Tests
- pytest tests for new logic, at the public function level, with literal expected values.
- External services (S3, APIs, models) mocked at the boundary only; fixtures small and realistic.
