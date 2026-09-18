# Native validation

Validated on September 17, 2026 on macOS arm64 with Python 3.14.6,
MLX/MLX Metal 0.32.2 and Transformers 5.17.0. Review: self-review.
Base: `28e9ccd9cb52d1d4a6f674b7914bae2de49d9aad`.

The updated patch contains the original production fix, the numeric regression
suite with its required copyright header, and one formatter-boundary test.
Three files: 179 insertions, 4 deletions. No dependencies or model code changed.

## Passed

From the upstream checkout, using the adjacent `.venv` created with
`uv venv --python /opt/homebrew/bin/python3 .venv` and populated with
`uv pip install --python .venv/bin/python -e 'upstream-mlx-lm[test]'`:

```sh
../.venv/bin/python -m unittest tests.test_qwen3_coder_numeric tests.test_tool_parsing tests.test_server.TestToolCallFormatter
uvx pre-commit run --files mlx_lm/tool_parsers/qwen3_coder.py tests/test_qwen3_coder_numeric.py tests/test_server.py
git diff --check
```

31 tests pass (exit 0). Black 25.1.0, isort 6.0.0 and Ruff 0.16.6 pass.
Explicit filenames include the untracked numeric suite; the earlier `--all`
hook invocation omitted it and incorrectly suggested complete lint coverage.

Previous baseline reproduction used normal package imports at the base SHA:
the 12 numeric test methods produced 26 assertion failures and 4 errors.
The new formatter test also failed against baseline. Both were independently
rechecked before this handoff update.

Additional checks: 30,018 generated decimal cases compared with a Fraction
oracle under decimal precisions 1, 3 and 28 passed. Formatter rejection and
recovery for six invalid values passed in streaming and non-streaming modes.
These are supplemental checks, not additional committed test methods.

The exported patch passes `git apply --check` against the clean baseline and
`git apply --reverse --check` against the validated working tree.

## Full discovery: environment errors, not a complete pass

Executed in each checkout:

```sh
HF_HUB_OFFLINE=1 TRANSFORMERS_OFFLINE=1 ../.venv/bin/python -m unittest discover tests/
```

Baseline: 232 tests, 22 errors, 1 skipped, exit 1.
Patched: 245 tests, 22 errors, 1 skipped, exit 1.
The complete error identifier lists match. All errors concern missing cached
Hugging Face model/tokenizer files. No new failures or errors were found.
Some errors occur during class setup, so these totals are not full suite coverage.
Logs are retained locally as `validation-baseline.log` and `validation-patched.log`.

Live inference, model-dependent HTTP tests and complete online integration
remain unvalidated. No model downloads were performed. No configured type
checker was found.

## Semantics and review notes

`9007199254740993` became `9007199254740992` on the baseline. The patch converts
the original text using Decimal, preserving the integer. Exact integrality
rejects fractions that binary float rounds to integers. The initial float check
preserves the finite-float admission bound. Zero is checked by its coefficient
so enormous zero exponents work without exponent expansion; nonzero underflow
is rejected. Number/float-schema behavior remains unchanged.

Integer infinities now raise ValueError instead of OverflowError. The formatter
catches ValueError, logs, and drops the invalid call; it can continue to format
a subsequent valid call. This does not redesign HTTP error behavior.

Earlier conversion-only microbenchmarks (best of five, 100,000 calls each)
measured baseline/patched microseconds per call: small integer 0.182/0.327,
large integer 0.266/0.407, integral decimal 0.189/0.345, number 0.230/0.231.
These are local parser timings, not inference throughput measurements.

GitHub API recheck confirms PR #1617 is closed and unmerged, closed on
2026-08-22. The earlier report calling it active was incorrect. PR #1417 is
merged and its float conversion is already in this base. The earlier search
found no active equivalent precision fix; search coverage is not a guarantee.

This is a technical handoff, not an upstream PR description. The human
contributor must understand the patch and author any upstream submission and
AI disclosure under the upstream contribution policy.
