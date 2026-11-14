# Reinforcement learning roadmap

V1 uses fixed question selection rules and records the decisions and outcomes needed to understand how they work. The live adaptive path uses fitted BKT and the trained supervised agent. The future experiment is a contextual bandit at one decision point per exercise. No Q-table or live learning is part of the v1 build.

## V1 logging

- Save one SelectionDecision per served item. Record the session position, policy/model/feature versions, eligible candidate IDs and difficulty levels, pre-answer context including mastery and recent behavior, risk scores when available, chosen question and difficulty, reason or fallback, and actual probability of the choice.
- A fixed deterministic choice has probability 1. The predicted risk is a separate value. Keep the actual selected question and its saved first answer, retry, hint and completion outcome linked. Use stable IDs and export only the data needed for evaluation.
- These logs describe the fixed policy and can support supervised evaluation or later retraining. They cannot show how a student would have done after an alternative difficulty that was never selected in the same circumstances.

## Later pilot

1. At one eligible position, choose between two safe difficulty options for the same skill, such as easier and medium. Apply all eligibility, coverage and session-length rules before offering alternatives. Record the available actions and their actual probabilities before observing the result.
2. Use current mastery, recent behavior, and candidate risk as context. Learn which choice leads to an independently correct answer on a comparable follow-up question on the same skill. Plan that follow-up within the fixed session count.
3. Add BanditOutcome when the pilot is implemented. Link the decision and follow-up SessionQuestion, store the reward definition version and outcome time. Use reward 1 for independent first-try correctness and 0 for an observed failure. Leave a missing follow-up unobserved.
4. Compare with the v1 fixed policy. Define the pilot and evaluation before switching the live selector. Keep agent retraining separate from updating the policy.

[Contextual bandit evaluation study](https://arxiv.org/abs/1003.5956)
