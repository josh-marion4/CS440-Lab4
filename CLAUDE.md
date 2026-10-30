# Vacuum Robot RL (CS440 Lab 4)

Project memory for Claude Code. Kept short on purpose: only rules that apply to every task.
Longer, task-specific context lives in `.claude/context/` and is imported on demand.

## Project
- A vacuum robot learns to reach a recharge station on a 4x4 grid with reinforcement learning.
- The robot does NOT know the transition model T or the reward function R.
- Main code: `CS440_LAB4.ipynb`. Logs: `logs/`. Q-learning is added in a later lab.

## Grid (row, col), start top-left
```
start  0   0   0
  0    X   0   0
  0    0   0   0
  0   -1   0  10
```
- X = obstacle (1,1). Trap = (3,1), reward -1. Charger = (3,3), reward +10. Both are terminal.
- Moves: up/down/left/right. 80% intended, 10% slip to each side. Walls and X keep the robot in place.
- Defaults: gamma = 0.9, living reward = -0.04.

## Rules
- Agent code may only use `env.reset()`, `env.step(a)`, `env.actions(s)`, `env.states`, `env.terminals`.
  Never read underscore attributes (`env._grid`, `env._living_reward`, `env._rng`). That would be cheating: it gives the agent T and R.
- Match the Lab 2 / Lab 3 notebook style (see `.claude/context/notebook-style.md`).
- Reuse `MOVES`, `SLIPS`, `show_grid`, and the path-log table instead of writing new versions.
- Seed every random run so results are reproducible.
- For review or theory questions, delegate to the sub-agents in `.claude/agents/`.

## Context imports
Lines below are added automatically by `scripts/context-switcher.sh` (UserPromptSubmit hook).
Each import is added at most once.

