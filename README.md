# Vacuum Robot RL — CS440 Lab 4

A vacuum robot on a 4x4 grid must find its recharge station without knowing the transition model
(T) or the reward function (R). This repository contains the environment and exploration code,
and is configured for Claude Code with two context engineering strategies:

- **Strategy 1 — Write (persistent memory):** project memory, user memory, and dynamic imports injected by a hook.
- **Strategy 4 — Isolation via sub-agents:** three specialist agents, each with a filtered context and restricted tools.

## The environment

```
start  0   0   0
  0    X   0   0
  0    0   0   0
  0   -1   0  10
```

| Item | Value |
|---|---|
| Start | (0, 0) |
| Obstacle | (1, 1) |
| Trap (terminal) | (3, 1), reward -1 |
| Charger (terminal) | (3, 3), reward +10 |
| Moves | up, down, left, right; 80% intended, 10% slip to each side |
| Living reward | -0.04 per step |
| Discount gamma | 0.9 |

## Project structure

```
vacuum-rl-lab4/
├── CLAUDE.md                         # S1: project memory (small, always loaded, shared via git)
├── CS440_LAB4.ipynb                  # Lab code: hidden-model environment + exploration
├── README.md                         # This file
├── requirements.txt                  # pandas, jupyter
├── .claude/
│   ├── settings.json                 # S1: registers the UserPromptSubmit hook
│   ├── context/                      # S1: dynamic imports, loaded only when relevant
│   │   ├── environment.md            #     env contract, rewards, known results
│   │   ├── rl-theory.md              #     Q-learning, exploration, regret (from slides)
│   │   └── notebook-style.md         #     Lab 2 / Lab 3 notebook conventions
│   └── agents/                       # S4: sub-agents (YAML frontmatter + prompt)
│       ├── env-validator.md          #     checks env against spec, read-only
│       ├── rl-theory-researcher.md   #     theory answers, no edit tools
│       └── notebook-style-reviewer.md#     style review, smaller model
├── scripts/
│   └── context-switcher.sh           # S1: picks context by branch/prompt, no duplicate imports
├── docs/
│   ├── DESIGN_DECISIONS.md           # Why each config file exists (video outline)
│   └── user-memory/CLAUDE.md         # S1: user-level memory, copy to ~/.claude/CLAUDE.md
└── logs/                             # CSV output from the notebook
    ├── lab4_log_episode1.csv … episode3.csv
    └── lab4_samples.csv              # every (s, a, r, s') sample for Q-learning
```

## Strategy 1 — Write (Persistent Memory Architecture)

| Layer | Where | Loaded |
|---|---|---|
| Project memory | `CLAUDE.md` | Every session, for everyone who clones the repo |
| User memory | `~/.claude/CLAUDE.md` (example in `docs/user-memory/`) | Every session, only on my machine |
| Dynamic imports | `.claude/context/*.md` | Only when the branch or prompt calls for it |

On each prompt, `scripts/context-switcher.sh` reads the git branch and the prompt text.
When a topic matches (for example "epsilon" → `rl-theory.md`), it checks `CLAUDE.md` with
`grep -qxF`. If the `@import` line is missing it appends it and prints the file so Claude sees it
right away; if the line is already there it does nothing, so imports are never duplicated.

| Trigger | Context added |
|---|---|
| Branch `env*` / prompt mentions grid, slip, reward, trap, charger | `environment.md` |
| Branch `qlearning*` / prompt mentions Q-learning, alpha, epsilon, explore, policy | `rl-theory.md` |
| Branch `docs*` / prompt mentions notebook, cell, style, log, csv | `notebook-style.md` |

## Strategy 4 — Isolation via Sub-Agents

| Sub-agent | Job | Tools | Model |
|---|---|---|---|
| `env-validator` | Verify the environment matches the spec and agent code never reads hidden T/R | Read, Grep, Glob, Bash | sonnet |
| `rl-theory-researcher` | Answer RL theory questions from course material | Read, Grep, Glob | sonnet |
| `notebook-style-reviewer` | Review changed cells against the Lab 2/3 style | Read, Grep, Glob | haiku |

The main session acts as the orchestrator and delegates by each agent's `description`.
Each agent receives only what its task needs, which prevents context confusion.

## Running

```bash
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace CS440_LAB4.ipynb
```

`docs/DESIGN_DECISIONS.md` explains each configuration file against the four questions from the
"Vague Prompts" slide: Audience, Context, Format and Constraints. The hook needs `bash` and either
`jq` or `python3`.

## Results so far (seed 0, 500 random episodes)

| Outcome | Episodes |
|---|---|
| Reached charger | 135 |
| Fell in trap | 256 |
| Timed out (50 steps) | 109 |

The robot's samples recover the hidden model: `right` from (2, 2) lands on (2, 3) 82% of the time.
Q-learning will be added in the next lab, using `logs/lab4_samples.csv` style transitions.
