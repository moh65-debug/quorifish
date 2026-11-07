# Wallz 2200 five-hour trainer

This package tests changes to the Wallz 2200-profile search engine embedded in Quoridor_Lab_Wallz.html. It tunes existing search options; it is not neural-network training or Stockfish Fishtest. It does not connect to KA or guarantee an improvement, a particular number of games, or a win against KA.

## Install in GitHub

1. Disable or delete the previous `.github/workflows/train.yml`, which trains a different engine and uses scripts replaced by this package.
2. Extract this ZIP. Upload the contents of the `wallz-training` folder into the root of your default branch. Include the hidden `.github/workflows/wallz-train.yml`, `training/` folder, and `Quoridor_Lab_Wallz.html`. Replacing previous `training/run.cjs` and `training/marathon.cjs` is intentional.
3. In Settings > Actions > General > Workflow permissions, enable Read and write permissions. Branch protection can still prevent the push.
4. Open Actions > Wallz five-hour training > Run workflow on the default branch.
5. After completion, download `trained/Quoridor_Lab_Wallz_champion.html` from the repository or run artifact to play offline. The champion settings are saved in `training-state/wallz-champion.json`, which the next run loads from the default branch.

Scheduled runs begin at 00:13, 06:13, 12:13 and 18:13 UTC, when GitHub schedules them. Scheduled jobs can be delayed or skipped. Disable this workflow in Actions to stop it. It can consume roughly 20 runner-hours a day; review your usage and billing. Up to four workers are used according to available CPUs.

## Experiments and persistence

The initial profile uses depth 6, wall bias 0.55 and no deliberate blunders. The offline app initially gives it five seconds per move. Experiments use 30 milliseconds per move to play more games. Good results at this short time limit need not transfer to five-second play.

Workers try one change to wall bias, allShortestPaths, fastWallGen or trapWalls, and compare candidate and champion in 20 paired games with reversed colors and two simple opponents. Unfinished pairs do not count. A candidate must win a strict paired test with at least 10 decisive pairs and pass the opponent gate. Correlated openings, timing noise and repeated tests can still lead to false positives. The workflow cannot invent a new search algorithm or learn a neural network. It may produce no promotions.

The runner saves a local checkpoint and playable HTML after each completed round. When five hours end, it stops incomplete experiments, uploads the current results and commits the champion for the next run. Incomplete experiments are discarded. A cancellation or hard runner failure can lose work that has not been uploaded or committed. If the default branch changes during the job, or a push is blocked, download the artifact and restore `training-state/wallz-champion.json` and `trained/Quoridor_Lab_Wallz_champion.html` manually. Test against KA separately; the workflow does not establish KA strength.

The bundled Wallz code came from the supplied website files. Verify redistribution permission before publishing it in a public repository.

## Local checks

With Node.js 22, in the extracted folder:

    node training/check.cjs
    node training/marathon.cjs --seconds=18 --workers=2 --pairs=1 --gate=1 --ms=5 --seed=11

These test mechanics, not playing strength. Delete local `training-state/`, `results/` and `trained/` after a quick test if you wish to start fresh. The GitHub workflow uses `--seconds=18000 --workers=4 --pairs=20 --gate=2 --ms=30`.
