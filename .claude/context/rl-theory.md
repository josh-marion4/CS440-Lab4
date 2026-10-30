# RL theory context (loaded when a prompt mentions Q-learning, alpha, epsilon, explore, policy or TD)

From the course slides (RL-L1, RL-L2, adapted from ai.berkeley.edu).

## Setting
- Still an MDP: states S, actions A, model T(s,a,s'), reward R(s,a,s'), looking for policy pi(s).
- New twist: T and R are unknown, so the agent must try actions and learn from samples (s, a, r, s').

## Q-learning (model-free, off-policy)
- Sample estimate: `sample = r + gamma * max_a' Q(s', a')`
- Running average: `Q(s,a) <- (1 - alpha) * Q(s,a) + alpha * sample`
- Converges to the optimal policy even when acting suboptimally, if the agent explores enough
  and alpha eventually becomes small (but not too fast).

## Exploration
- epsilon-greedy: with probability epsilon act randomly, otherwise act on the current Q.
  Lower epsilon over time so the agent stops thrashing once it has learned.
- Exploration function: `f(u, n) = u + k / n` adds a bonus to rarely visited states (n = visit count).
- Regret: total reward lost while learning. Exploration functions have lower regret than random actions.

## Comparison with Lab 3
- Lab 3 used value iteration because T and R were known. Expected Lab 3 utility at (0,0) with
  living reward -0.04 was 4.35; a learned policy should approach that average return.
