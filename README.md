# Quoridor Lab: automated engine experiments

This is a starter inspired by Stockfish's Fishtest approach: propose a change, play paired games, and keep it only if the results favor it. It is **not** Stockfish or the Fishtest service. It currently tunes one setting: the value the engine assigns to a remaining wall. It does not change its own algorithms, train a neural network, or guarantee it will beat Ka.

## Install in your GitHub repository

1. Extract this ZIP on your computer. Open the extracted `quoridor-training` folder.
2. On GitHub, open the repository you want to use. If you are starting fresh, create a repository. A public repository makes the code visible to everyone.
3. Upload the **contents** of the folder to the repository root, including the `.github` folder. The workflow file must be at `.github/workflows/train.yml` on the default branch.
4. Open the repository's **Actions** tab. Enable Actions if GitHub asks. Choose **Quoridor engine experiments**, then **Run workflow**.
5. Open the completed run. Download the **quoridor-results** artifact. Inside `results`, open `Quoridor_Lab_trained.html` in Chrome or Edge to use the selected engine offline. `summary.json` gives the comparison; `games.json` has the move records.

The workflow also runs daily at 03:17 UTC when scheduled workflows are enabled. Disable the workflow in the Actions tab if you want to stop scheduled experiments.

## What each run does

- Restores the latest cached champion or starts with wall value 55.
- Proposes a small change and runs 20 pairs (40 games). Both versions get the same position and a fixed search-node budget; they swap sides for the second game.
- Applies a conservative one-sided sign test to decisive pairs. At least 10 decisive pairs and a p-value below 0.01 are required for promotion. A tied pair contributes no evidence. Games stopped by repetition or a move limit are unfinished, and their whole pair is excluded; they are not claimed as official draws.
- Writes the champion and report, then saves a checkpoint for the next run. The downloadable HTML uses the champion's wall setting; the repository's original HTML is unchanged.

This is exploratory research, not a guarantee: repeated testing, a small number of openings, shared engine weaknesses, and adaptive candidate selection weaken statistical conclusions. These tests compare variants of the same engine, not Ka, Apex, or Ishtar. Further independent opponents and more games are needed before claiming an improvement in general play. The prior Lab engine lost to Ka in the limited games already tested.

## Checkpoints and cost

The next run restores the latest `quoridor-champion-` Actions cache. GitHub may evict caches after inactivity or storage pressure. Each run also publishes `champion.json` in its artifact for 30 days: download it for safekeeping. A fresh repository has no training history. Do not delete useful artifacts until you have downloaded them.

On public repositories standard hosted runners are currently free under GitHub's conditions. Private repository usage draws from your included usage and can incur charges. Check your plan, limits, storage, and GitHub policies. This workflow uses a standard Ubuntu runner and has a 120-minute per-run cap; GitHub may delay scheduled runs.

## Run locally for a quick check

Requires Node.js 22 or later:

```sh
node training/check.cjs
node training/run.cjs --pairs=1 --nodes=100 --seed=1
```

The quick check is far too small to show playing strength. For actual experiments the workflow uses 20 pairs and 800 nodes per search. You can change those settings in `.github/workflows/train.yml`; larger values take longer. `training-state` and `results` are generated locally and do not need to be committed.
