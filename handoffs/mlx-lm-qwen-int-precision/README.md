# MLX-LM contribution handoff

This branch is a staging area only. It does **not** modify `H-H-E/developer` main and it does not open or represent an upstream PR.

## What is here

- `mlx-lm-qwen-integer-precision.patch` — proposed three-file MLX-LM patch, including a formatter-boundary test.
- `VALIDATION.md` — native validation results and remaining integration limits.
- `MAC_VALIDATION_PROMPT.txt` — instructions for continuing validation on an Apple Silicon Mac.

Investigated upstream base: `ml-explore/mlx-lm@28e9ccd9cb52d1d4a6f674b7914bae2de49d9aad`.

The bug is in Qwen3-Coder tool argument parsing: integer-schema values are routed through Python `float`, so large exact integers such as `9007199254740993` can be silently changed before the tool receives them. The patch uses the original decimal text for exact integer conversion while retaining the existing finite-float admission range.

## Pick up on the Mac

```bash
git clone --single-branch \
  --branch handoff/mlx-lm-qwen-int-precision \
  https://github.com/H-H-E/developer.git mlx-lm-handoff

cd mlx-lm-handoff/handoffs/mlx-lm-qwen-int-precision

git clone https://github.com/ml-explore/mlx-lm.git upstream-mlx-lm
cd upstream-mlx-lm
git fetch origin
git switch -c fix/qwen3-coder-integer-precision origin/main

# First re-check current upstream issues/PRs and confirm the defect still exists.
git apply --check ../mlx-lm-qwen-integer-precision.patch
git apply ../mlx-lm-qwen-integer-precision.patch
```

Then give the coding agent `../MAC_VALIDATION_PROMPT.txt` and have it run the native/full-package validation before any upstream publication.

## Important

MLX-LM's contribution policy requires the human contributor to understand every line and explicitly disclose AI assistance. It also prohibits AI-written GitHub posts/PR descriptions. Do not have an agent publish a PR description or maintainer comment on your behalf.

No upstream write has been made from this handoff.
