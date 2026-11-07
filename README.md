# Quoridor Lab — tournament training v2

This project tests controlled changes to the offline Quoridor Lab engine. It is inspired by Stockfish's testing process, but it is not Fishtest, neural-network training, or proof of a win against Ka.

## What is new

Each run proposes a change to one of four settings: wall value, distance-to-goal value, the number of wall candidates searched, or the existing wall-vulnerability check. It tests the candidate against the saved champion in 20 pairs of games, swapping colors from the same starting position. Openings vary and some include legal walls.

Then both versions play the **same additional openings** against two fixed playing styles: a pawn racer and a greedy wall player. These are simple bots using the same rules implementation, not independent Quoridor engines. A candidate is kept only if it passes the direct comparison and the opponent checks. The tests are exploratory: small samples, adaptive repeated testing, deterministic play, and shared rules mean that a promotion does not establish an improvement against humans or Ka. Repetition or a move limit is recorded as unfinished, not as an official draw.

The workflow checks the rules, including a position from an actual loss to Ka where no wall can legally stop the winning route. It produces a game log, a summary, a checkpoint, and a ready-to-use offline app using the selected settings.

## Update the existing GitHub repository

1. Extract the ZIP and open the `quoridor-training-v2` folder.
2. Upload the folder's **contents** to the root of `moh65-debug/quorifish`, replacing the existing matching files. Include the `.github` folder; the workflow must be at `.github/workflows/train.yml` on the default branch.
3. In GitHub, open **Actions → Quoridor engine experiments v2 → Run workflow**.
4. Open the completed run and download `quoridor-v2-results-...`. Read `results/summary.json` and `results/games.json`. Open `results/Quoridor_Lab_trained.html` in Chrome or Edge for the offline app.

This version deliberately uses a new checkpoint cache, so it starts from the original v6 settings (wall value 55). It does not import the previous workflow's training history. The prior run you showed rejected its candidate and retained 55. Download artifacts for safekeeping; caches may be evicted and artifacts expire after 30 days.

The workflow runs daily at 03:17 UTC as well as manually. Disable the workflow in GitHub Actions to stop scheduled runs. The 120-minute job limit stops the job if exceeded. A failed or timed-out run does not update the champion checkpoint. Public standard runners are subject to GitHub's terms and limits; private repositories may incur usage charges.

## Interpreting results

`promoted: true` means the candidate passed this workflow's internal gates. It does not mean Ka was beaten. Fields under `gates` compare candidate and champion wins against each simple opponent; the gate tolerates at most one fewer win for the candidate per opponent. The direct pair test requires at least 10 decisive pairs and a one-sided sign-test p-value below 0.01. This threshold is not a long-term false-positive guarantee because candidates are tested repeatedly and games are not necessarily independent.

`results/Quoridor_Lab_trained.html` uses the **saved champion**, not an unsuccessful candidate. Before claiming progress against Ka, play a separate match against Ka, alternate sides, record its setting and Lab's thinking time, and use several games. This package does not automatically connect to or run matches against the remote Ka service.

## Local tests

Requires Node.js 22+:

```
node training/check.cjs
node training/run.cjs --pairs=1 --gate=1 --nodes=100 --seed=1
```

The small local run is a functional check only. The GitHub workflow uses 20 direct pairs, 4 starting positions per opponent, and a 1200-node budget. Every additional position is played twice per engine version (once per color). Increasing those settings increases runtime. `training-state` and `results` are generated output and should not be committed.
