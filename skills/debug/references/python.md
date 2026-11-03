# Python / AWS / data bug patterns

Check the ones that match the symptom. Confirm library behaviour against the installed versions (`pip show <package>`).

## Language
- Mutable default arguments (`def f(items=[])`) shared between calls.
- Late binding in closures and lambdas inside loops.
- Naive vs timezone-aware `datetime` compared or subtracted; `datetime.now()` vs `datetime.now(timezone.utc)`; dates parsed with the wrong day/month order.
- Float comparison and rounding (`round()` uses banker's rounding); money in floats.
- Exceptions swallowed by a broad `except Exception` or a bare `except`, hiding the real error.
- Generators or iterators consumed twice (the second pass is empty).
- Encoding: files read without `encoding="utf-8"`, BOMs in CSV headers, accented characters in names and transcripts.

## pandas and data
- `merge` / `join` that duplicates or drops rows: compare row counts before and after, use `validate="one_to_one"` or `"many_to_one"`, check `how=`.
- IDs read as floats (because of NaN) and losing leading zeros or turning into `1.0`: pass `dtype=` explicitly.
- NaN: dropped silently by `groupby` (unless `dropna=False`), never equal to itself, `fillna(0)` where 0 is a real value.
- Chained assignment and `SettingWithCopyWarning`; mutating a DataFrame another part of the code still uses.
- String keys with inconsistent case or whitespace on joins and filters.
- `apply` / `iterrows` producing different types than a vectorized version would.

## Models and ML
- A model pickled with one scikit-learn version loaded with another (`InconsistentVersionWarning`): pin versions or retrain.
- Preprocessing at inference that differs from training (different cleaning, feature order, tokenizer, sample rate).
- Non-determinism: unset seeds, GPU non-determinism, sampling temperature in LLM and ASR calls.

## Audio, ASR and LLM pipelines
- Wrong audio format reaching the model: sample rate, channel count, codec (opus, wav), duration limits, silent or truncated files.
- Looping or repeated output in ASR or LLM transcription: check the decoding parameters (temperature, repetition or no-repeat settings, condition on previous text), chunk boundaries, silence, and the model's documented limits.
- Prompt or template changes that silently change output format; parsing model output without validating it.
- Text matched against filenames or ids with different normalization.

## AWS
- S3 `list_objects_v2` returns at most 1000 keys per call: use a paginator.
- Key prefixes and paths: leading or double slashes, URL-encoded characters in event keys (S3 event notifications URL-encode the key).
- Lambda: globals reused between warm invocations (state leaks, cached clients with expired credentials), only `/tmp` is writable, timeout and memory limits, the handler retried on error (idempotency).
- IAM `AccessDenied` that looks like `NoSuchKey`, region mismatches, credentials from the wrong profile locally.

## Streamlit
- The script reruns top to bottom on every interaction: state not kept in `st.session_state` is lost, expensive work not cached with `st.cache_data` / `st.cache_resource` reruns every time.
- Widgets without a stable `key` reset or collide.

## Tools
- `python -X dev` to surface warnings, `pdb` / `breakpoint()`, `logging` at DEBUG for the module involved.
- `cProfile` / `pyinstrument` for slowness, `tracemalloc` for memory.
