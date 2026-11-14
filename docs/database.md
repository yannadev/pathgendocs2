# Pathgen database

Use Django models and migrations on SQLite in development and PostgreSQL in production. Use stable primary keys, timezone-aware timestamps, and explicit constraints for ownership and lifecycle. Use ordinary relational columns for queries and JSONField for immutable feature snapshots. Set the custom `AUTH_USER_MODEL` before the first migration.

## Accounts, classes, curriculum

| Model / app | Fields and rules |
|---|---|
| User / accounts | Django AbstractUser with unique username, hashed password, display_name, active state, and explicit admin/teacher/student application role. Account ID remains stable across profile changes. |
| Classroom / classrooms | name, teacher_id, created_at. Required teacher is a teacher-role account. One teacher may own many classes. |
| StudentProfile / classrooms | Unique user_id, required classroom_id. The linked account must be a student. Create the user and profile together. Protect references to historical rows. |
| Lesson / curriculum | Unique code, title, sort_order, named route or template key, content availability status. Use the six entries in [system.md](system.md). |
| Skill / curriculum | Unique stable code, display name, skill order and canonical teaching lesson. Use the ten codes in [system.md](system.md). |
| LessonSkill / curriculum | Unique lesson and skill pair, scope (taught or cumulative review). Only Expressions and Applications has an approved operation review scope. |
| SkillPrerequisite / curriculum | Skill and prerequisite skill pair with unique pair, no self link, review section on the prerequisite's canonical lesson. Validate links. |

Cross-table role and ownership checks belong in application services. For example, verify the assigning teacher owns the selected classroom and that a question's skill is allowed by its lesson and revision scope. Data seed operations must be repeatable by stable codes.

## Teacher content and student practice

| Model / app | Fields and rules |
|---|---|
| Exercise / practice | owner_teacher_id, lesson_id, status and active published revision pointer. Use the stable exercise ID for attempt limits across revisions. |
| ExerciseRevision / practice | exercise_id, revision number, title, target question count, practice scope, draft/published state, published_at. Unique exercise/revision pair. Store required skills as a relation, with unique revision/skill pairs. Freeze published pool, choices, media, hints, skill tags, and required skills. Create a new draft to edit them. |
| Question / practice | revision_id, stable item key within exercise, author order, format, prompt, skill_id, difficulty tag, hint, and media. Text has no media. Figure stores validated file reference and description. Video stores normalized video ID and start seconds. Keep the stable item key for the same item across revisions; a materially different replacement can have a new key. |
| AnswerChoice / practice | question_id, position 1–4, text, is_correct. Unique position per question and at most one correct choice. Validate exactly four choices and exactly one correct before publication. |
| ExerciseAssignment / practice | exercise_id, classroom_id, assignment status and publication metadata. Unique exercise/classroom pair. Active assignments govern new sessions. Unpublish disables new starts without deleting history. |
| ExerciseSession / practice | student_id, exercise_id, revision_id, classroom_id at start, attempt_number 1 or 2, status, target_count, started_at and completed_at. Unique student/exercise/attempt_number. Session 2 requires completed session 1. Resume the saved session and revision. |
| SessionQuestion / practice | session_id, revision question_id, position, shown_at, completed_at, hint_revealed_at, finalized timing fields and outcome. Unique session/position and session/question. Keep answer duration and video viewing duration separately; represent unavailable timing as null. |
| AnswerAttempt / practice | session_question_id, attempt_number 1 or 2, choice_id, is_correct, submitted_at. Unique session_question/attempt_number. Check that the choice belongs to the served question and retry follows an incorrect first submission. |

Keep saved attempts and published revisions immutable. Use protection for historical references and retain files used by published questions. An exercise can be deleted only if never published. Grade on the server. Save first-try and corrected-on-retry counts from persisted answers at completion, or derive them consistently from the same records. Report progress from class-scoped assignments, sessions and assessment attempts.

## Mastery, decisions, feedback

| Model / app | Fields and rules |
|---|---|
| StudentSkillMastery / practice | student_id, skill_id, mastery_probability in [0,1], evidence_count, bkt_version, updated_at. Unique student/skill. Initialize lazily from the fitted prior. |
| MasteryUpdate / practice | Unique session_question_id, matching first_answer_id, skill_id, prior, posterior after observation and next mastery, observation, BKT version, created_at. One event per first answer. |
| SelectionDecision / practice | Unique served SessionQuestion, timestamp, policy/BKT/agent/feature schema/difficulty mapping versions, eligible candidate snapshots (question, skill, difficulty, prediction if scored), exact 12 selected inputs when model-ready, chosen action and actual selection probability, fallback reason when applicable. Save a separate readiness flag and nullable missing feature values. Record history as it existed before the selected question was answered. |
| ExerciseFeedback / practice | Unique session_id, focus skill, strength if present, review destination if available, generated or fallback text, status/source, and model/knowledge versions when used. Write a deterministic summary when the session completes and let the local generation step replace only its prose if available. |

For v1, keep contextual bandit logging in `SelectionDecision`; future reward linkage belongs to [r_learning.md](r_learning.md). The supervised export joins the selected snapshot to the served question and attempts; use the stable student ID or a pseudonym and chronological order for grouping, not as predictors. Do not store copies of the Junyi CSVs in the app database.

## Assessments

| Model / app | Fields and rules |
|---|---|
| AssessmentForm / assessments | kind (pretest or posttest), revision number, title, state, published_at. Unique kind/revision. Keep at most one active published form per kind. Protect published versions and existing attempts. |
| AssessmentItem / assessments | form_id, position, skill_id, prompt, four options and one correct index. Unique form/position. Validate both forms cover all ten skills with comparable counts and challenge. |
| ClassAssessmentRelease / assessments | classroom_id, posttest_form_id, opened_by, opened_at. One release per classroom for the v1 unit. Pin the published form on release. |
| AssessmentAttempt / assessments | student_id, kind, pinned form_id, class_id at start, state, timestamps, correct_count and item_count on submission. Unique student/kind for the v1 unit. |
| AssessmentAnswer / assessments | attempt_id, item_id, selected option, last_saved_at and server correctness after final submission. Unique attempt/item. Lock answers after submission. |

The pretest unlock is a submitted pretest attempt. Posttest start requires an eligible student's class release and a submitted pretest. Expose scores after submission and report score change in percentage points. Use practice and assessment tables independently.

## Write boundaries and acceptance

- First answer, MasteryUpdate and StudentSkillMastery change commit together. Serialize concurrent updates for a student/skill and check duplicates within the transaction. Finalizing a question and advancing must converge on one persisted next position/decision. Preserve the saved answer on replay. Use short transactions, common lock order, and uniqueness constraints. PostgreSQL can lock rows; SQLite uses its own write serialization and may need immediate transactions in development.
- Make start, retake, final submit, open-posttest, and final feedback writes safe to repeat. Do local inference outside answer writes. If candidate scoring happens outside a write transaction, recheck the saved position and the selection snapshot or reuse the result already written before committing a next question.
- Index the report and history access paths, especially student/skill, student/session timestamps, classroom assignments, exercise sessions by class, and assessment attempts by student and kind. Check probabilities, ordinal ranges and durations in models/services where appropriate.
- Verify direct access checks, completion and retry counters, resume, revision stability, and duplicate/concurrent requests. Run the concurrency acceptance checks on PostgreSQL before deployment.

[Django constraints](https://docs.djangoproject.com/en/5.2/ref/models/constraints/) · [Django transactions](https://docs.djangoproject.com/en/5.2/topics/db/transactions/) · [Django database backend notes](https://docs.djangoproject.com/en/5.2/ref/databases/)
