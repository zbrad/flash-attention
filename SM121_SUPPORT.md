# sm_121 (GB10) support in this fork

Upstream Dao-AILab/flash-attention's `setup.py` does not yet build a native
sm_121 (GB10, "Spark") target on `main` as of this fork's base commit
(`0251105`, 2026-08-26). Confirmed via GitHub search: issue
[#1969](https://github.com/Dao-AILab/flash-attention/issues/1969) tracks
the gap, and PR
[#2257](https://github.com/Dao-AILab/flash-attention/pull/2257) ("Add
sm_121 support for NVIDIA GB10 GPUs (CUDA 13.0+)") is an open, mergeable,
single-file fix for it that had not been merged yet.

## What was done

Cherry-picked PR #2257's commit
(`54ea03dfc21760a859f0e7a05caec7249ba5b47b`) onto this fork's
`tuned-builds` branch. It:

- Adds `121` to `cuda_archs()`'s default `FLASH_ATTN_CUDA_ARCHS` list.
- Adds a `bare_metal_version >= 13.0 and "121" in archs` branch to
  `add_cuda_gencodes()` emitting `-gencode
  arch=compute_121f,code=sm_121` — family-specific PTX (Blackwell 12.x
  family compat) compiled down to native `sm_121` SASS.
- Slightly restructures the neighboring Thor (`sm_110`/`sm_101`) branch
  (unrelated cleanup bundled in the same upstream commit, kept as-is
  rather than partially cherry-picked).

Verified before applying: `nvcc -gencode arch=compute_121f,code=sm_121`
compiles a trivial `.cu` file successfully against this box's CUDA 13.3
toolkit.

## Requires CUDA >= 13.0

`tuned/env.sh` fails loudly on `gb10` if `CUDA_VERSION_COMPACT < 130`
rather than silently falling through to a gencode-less build.

## Not upstreamed

This is a local cherry-pick, not a PR back to Dao-AILab (#2257 already
exists and is out of this fork's hands to merge). If #2257 lands upstream,
a future `git fetch upstream && git rebase upstream/main` on
`tuned-builds` should drop this commit cleanly (identical diff).
