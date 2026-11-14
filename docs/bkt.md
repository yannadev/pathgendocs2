# Bayesian Knowledge Tracing

BKT keeps a mastery probability for each student and each of the ten skills. Pathgen updates the primary skill once from the first submitted answer to each tracked practice question. It uses fitted priors for new skills. Assessment responses, untracked lesson interactions, hints and retries do not add observations.

## Fitted artifact

- Runtime path: `model_artifacts/bkt_parameters.json`.
- Verified handoff: `C:/Users/jhadz/Downloads/path.dev/model/bkt_param/bkt_parameters.json`. The copy in `C:/Users/jhadz/Downloads/roror/bkt_parameters.json` has the same SHA-256.
- SHA-256: `cd194bb8ae2090150685e91687a338f0d47ee2b884b64f11e365d6f9db33d6c5`.
- Artifact schema version 1. Read `skills[skill_id].initial_mastery`, `learn`, `guess`, and `slip`. Forgetting is zero. Require all ten expected codes and valid probabilities. For the real adaptive integration, surface missing or mismatched fitted parameters as configuration errors rather than inventing values.

The fit used 263,813 Junyi responses, capped at 30,000 per skill, with complete chronological student histories for each retained skill. On 3,921 held-out Junyi students, mean per-skill validation log loss was 0.446 versus 0.494 for the constant baseline. The opposites guess parameter was near the lower bound. Junyi's correctness flag incorporates a hint override. Pathgen offers a hint only after an incorrect first answer, and uses that original answer once. Local validation is needed for transfer.

## Update

Let p be the student's current mastery, g the guess rate, s the slip rate and t the learning rate for this skill.

```text
correct:    known = p * (1 - s), unknown = (1 - p) * g
incorrect:  known = p * s,       unknown = (1 - p) * (1 - g)
posterior = known / (known + unknown)
next_mastery = posterior + (1 - posterior) * t
predicted_correctness_before_answer = p * (1 - s) + (1 - p) * g
```

Store prior, observation, posterior, next mastery and parameter version in the one MasteryUpdate event. Store the next mastery and evidence count in StudentSkillMastery. Follow the transaction in [database.md](database.md). For an eligible candidate, the agent uses the latest already-saved mastery for the candidate's skill, or its fitted prior if there is no practice evidence. A retake uses current mastery and adds new observations from its first answers. Keep the BKT version compatible with the agent bundle. Re-evaluate the pair before changing either artifact.

Verify known numeric examples, exact use of fitted priors, one update after a wrong first answer and later correct retry, unchanged mastery for other skills, and correct replay behavior on duplicate submission.
