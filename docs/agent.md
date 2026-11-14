# Supervised struggle prediction agent

The agent returns the probability that a student will struggle with a candidate question. The fixed rules in [special_algo.md](special_algo.md) choose the question. The model was trained offline and is loaded for inference in v1.

## Runtime artifact and interface

- Use `model_artifacts/agent_model.pkl`. Verified source: `C:/Users/jhadz/Downloads/path.dev/model/agent_model/agent_model.pkl`. The copy in `C:/Users/jhadz/Downloads/roror/agent_model.pkl` matches.
- SHA-256: `b31a28a36e0e14a49a298eecd29bf915465f6d3f805ffb9fc6ab5a04dc30a091`. Load only the approved artifact after verifying its checksum. A pickle is executable on load. Never accept a model upload from an app user.
- The trusted bundle has `schema_version`, `model`, `feature_columns`, `target`, `positive_class`, `decision_threshold`, `calibration`, `sklearn_version`, `bkt_parameters_sha256`, and evaluation metadata. Verify schema version 1, the feature order below, target, positive class 1, and the linked BKT artifact. Use the pinned runtime in [tech_stack.md](tech_stack.md).
- Build a pandas DataFrame with the saved feature order. Call the saved `model.predict_proba` and take the column for class 1. The fitted pipeline and calibrator already implement preprocessing. Pass missing numeric durations as NaN in memory and null in persisted JSON. Reject nonfinite output or probabilities outside [0,1]. The saved 0.315 classification threshold is for the Junyi evaluation. Use probabilities for question selection.

## Exact feature contract

Compute candidate features before answering the selected question. History means the student's earlier completed tracked question encounters across exercises, in completion order with a stable tie break. Include the most recently completed encounter when selecting the next question. An encounter can include a retry but contributes one row to the history.

| Order | Feature | Pathgen source |
|---|---|---|
| 1 | `candidate_skill_id` | Candidate's primary skill code. |
| 2 | `candidate_difficulty` | Versioned numeric mapping for the candidate's teacher difficulty tag. Training used a Junyi template's historical struggle rate. |
| 3 | `last_correct` | Previous encounter's first-try correctness as 0 or 1. |
| 4 | `last_attempt_count` | Previous encounter's number of submitted attempts. |
| 5 | `last_hint_used` | Whether the hint was revealed in the previous encounter as 0 or 1. |
| 6 | `last_duration_sec` | Previous encounter's recorded active answering seconds, including its retry when applicable. |
| 7 | `recent_correct_rate` | Mean first-try correctness across up to the last five completed encounters. |
| 8 | `recent_hint_rate` | Mean hint use over that window. |
| 9 | `recent_mean_attempts` | Mean submitted attempts over that window. |
| 10 | `recent_median_duration_sec` | Median of valid durations in that window. Use missing if none are available. |
| 11 | `candidate_skill_prior_encounters` | Number of earlier completed tracked encounters on the candidate skill. |
| 12 | `candidate_skill_bkt_mastery` | Current BKT estimate for the candidate skill or its fitted prior. |

**V1 defaults:** start the first-ever question in teacher pool order because the fitted model requires prior history. A session can use earlier completed practice from other sessions. Measure active question time and keep video viewing time separate, excluding playback from the duration feature where it can be measured reliably. Use missing timing when an interruption, invalid timer or playback state prevents a reliable measurement. Record the feature schema version. Timing and difficulty scales require validation on Pathgen.

Use provisional configurable numeric difficulty values Easy 0.15, Medium 0.30, Hard 0.50 for the first pilot, version the mapping, and report them as uncalibrated. A missing or invalid mapping invokes the selector fallback. Re-estimate the mapping using Pathgen practice from training students only when there is enough evidence, and evaluate on different students.

## Data provenance and later retraining

The Junyi problem log supplies eight source fields: `user_id`, `time_done`, `exercise`, `problem_type`, `correct`, `count_attempts`, `hint_used`, `time_taken`. The external skill map matches `exercise` and `problem_type` to the ten modeled skills. The original exercise catalog validates mapped exercise names through log `exercise` → catalog `name`. Training used 100,000 rows, validation 25,000, and test 30,000, balanced across skills and split by student. Training BKT and difficulty features use five student folds. Validation and test use frozen training estimates.

On held-out Junyi students the agent had 80.11% accuracy, 50.64% struggle recall and 0.4162 log loss. The BKT-only proxy had 80.27%, 40.48%, and 0.4342 respectively. These are Junyi results. Pathgen performance must be measured on its own questions and students.

Export later from SelectionDecision joined to the selected SessionQuestion and AnswerAttempt. Save the selected feature snapshot at selection time even if inference is unavailable but all feature inputs exist. Attach `struggle_on_next_question = 1` if first-try correctness is false, there is more than one submitted attempt, or the hint was used. Otherwise use 0 for a completed question. Omit unserved candidates, unfinished questions, and rows without usable history features. Include pseudonymous student key and chronological metadata for grouping. They stay outside the predictor inputs. Compare any recalibrated or retrained model against the frozen Junyi model on the same held-out Pathgen students. Evaluate by skill and report accuracy, struggle recall and probability quality. Use this export offline. See [r_learning.md](r_learning.md) for the separate question selection experiment.

Keep the fitted version in deployment until a replacement has passed evaluation and the same runtime checks. The v1 web application records the data for this work. Offline retraining and switching the live model are separate tasks.

[Scikit-learn model persistence](https://scikit-learn.org/stable/model_persistence.html)
