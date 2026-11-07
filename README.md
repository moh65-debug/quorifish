# Quoridor five-hour engine experiments

This package runs bounded, repeated Quoridor engine tournaments using up to four CPU workers. It is a testing and parameter-tuning system, not neural-network training and not the Stockfish Fishtest service. It does not connect to Ka and cannot guarantee a stronger engine, a win against Ka, a specific number of games, or a specific number of improvements.

## Install in moh65-debug/quorifish

1. Extract the ZIP and open the `quoridor-training-endurance` folder.
2. Upload its **contents** to the root of the repository's default branch, replacing matching files. Include `.github/workflows/train.yml`, the files in `training/`, and `Quoridor_Lab.html`. This replaces the earlier daily workflow.
3. In GitHub, check **Settings → Actions → General → Workflow permissions**. The workflow needs permission to write repository contents. Branch protection can still prevent its push.
4. Open **Actions → Quoridor five-hour experiments → Run workflow**. Ensure it starts on the default branch.
5. After completion, the workflow commits `training-state/champion.json` and `trained/Quoridor_Lab_trained.html` to the default branch. Download the latter to use the chosen engine offline. The run's artifact also contains `results/run-summary.json`, `results/experiments.jsonl`, the HTML, and the checkpoint.

A schedule also starts runs at 00:13, 06:13, 12:13, and 18:13 UTC. Each run works for up to 4 hours 55 minutes, leaving time for saving and pushing. GitHub may delay or skip scheduled runs, and only one run in the concurrency group runs at a time. To stop repeated runs, disable this workflow in the Actions tab. This schedule can use nearly 20 runner-hours per day; review your plan, usage, and GitHub's policies. Standard runners for public repositories are currently free subject to limits and terms; private usage may be billed.

## How the tournament works

At the beginning, the job reads the committed champion checkpoint. If it is absent, the workflow attempts to restore the previous v2 Actions cache. If neither exists it starts with wall 55, race 100, branch 10, and the vulnerability check enabled.

Up to four workers independently test changes to one of those four settings. Each experiment plays 20 direct pairs with colors reversed and 32 games against two simple playing styles. All games use 1,200 search nodes per move; seeds produce varying openings. A game reaching the repetition or move limit is recorded as unfinished rather than an official draw.

The candidate must pass the direct and opponent gates from v2, and additionally meet a stricter p-value threshold of 0.01 divided by the number of workers. If several pass the same round, one is selected by worker order; all other results are retained in the run report. Each new round tests against the selected champion, not against a stale snapshot.

These are exploratory tests. Repeated testing, the small pool of simple opponents, potentially correlated openings, and equal-node rather than real-time play prevent the p-value threshold from guaranteeing long-term improvement. The system cannot invent new search algorithms or learn a neural network. Play an independent match against Ka to assess the target; no remote Ka connection is included.

## Saving and recovery

The champion is updated on the runner after each completed round. The final state is published as an artifact before the push. The workflow refuses to push if the default branch changed during the five-hour run; it never force-pushes. If permissions, protection rules, a branch change, or a timeout prevent the commit, download the artifact before it expires and restore the champion manually. GitHub-hosted machines are temporary: a canceled run or hard timeout can lose work that was not uploaded or committed. Artifacts expire after 30 days.

The original `Quoridor_Lab.html` is the fixed source template. The current playable result lives in `trained/Quoridor_Lab_trained.html`. Do not manually alter the template's engine strings without updating and testing `training/run.cjs` and `training/marathon.cjs`.

## Local quick check

With Node.js 22 or later:

```
node training/check.cjs
node training/marathon.cjs --seconds=10 --workers=2 --pairs=1 --gate=1 --nodes=100 --seed=1
```

The quick check exercises mechanics only, not playing strength. The full workflow runs `--seconds=17700 --workers=4 --pairs=20 --gate=4 --nodes=1200`.
