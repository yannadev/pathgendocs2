# Pathgen build order

Open Codex in the Pathgen repository root, with these 13 Markdown files in its `docs/` folder. Then say **“Execute phase 1 from docs/build_order.md.”** For another phase, substitute its number. This file and [system.md](system.md) are the starting point. The referenced specifications supply the product decisions and implementation contracts.

## Execution rules

1. Read this phase, [system.md](system.md), the named specifications, and the repository's `AGENTS.md` if present. Inspect the current tree and check the preceding phase's acceptance criteria. Preserve working code and existing user changes.
2. Implement the requested phase and necessary unfinished prerequisites within the same scope. Report an external dependency if it is unavailable, and continue with independent tasks. Use the spec defaults unless the user has changed them in the repository. Treat optional dialogs and future experiments as outside the base build.
3. Verify the phase through its normal entry points and run the checks listed under **Done when**. Record results, remaining gaps, and exact commands in the handoff. Mark the phase complete only when its checks pass. Keep demonstration data clearly labeled and separate from real student records.

## Phase 1 — Bootstrap Django, Docker, and the repository

Read [tech_stack.md](tech_stack.md), [file_structure.md](file_structure.md), and the Accounts section of [database.md](database.md).

1. Create or reconcile the project skeleton and six app packages. Add `manage.py`, WSGI, root URLs, split settings, a readiness route, and Django template and static settings. Set up the custom User model and `AUTH_USER_MODEL` before running the initial migration. If an existing project already migrated a different user model, handle the existing schema deliberately before migrating.
2. Pin the base Python/Django dependencies, add the Django Dockerfile and development Compose configuration, and define the SQLite and upload volumes. Configure environment loading, development and production settings, and the example environment. Production settings read PostgreSQL credentials and fail clearly if missing.
3. Add the base template and plain CSS/JS entry points. Put base styles outside media queries and include the six commented breakpoint definitions from [tech_stack.md](tech_stack.md#mobile-first-css) in the shared stylesheet. Activate only the overrides a component needs. Add the named Google Fonts, Chart.js and AnimateIcons browser dependencies where needed, with no Node build. Write the README and repository root `AGENTS.md`, with the latter pointing to `docs/system.md` and `docs/build_order.md`.
4. Run the initial migrations and document the exact clean clone startup and admin bootstrap steps. If these specs were supplied outside the repo, place the 13 active files in repo `docs/` before implementation.

**Done when:** a clean checkout builds and starts via `docker compose up --build`, the health route responds, `python manage.py check` and initial migrations pass inside the container, SQLite survives a restart, the frontend build runs without npm, and the shared stylesheet contains the six specified breakpoints.

## Phase 2 — Accounts, classes, and curriculum

Read [user_features.md](user_features.md), [database.md](database.md), [pages.md](pages.md), and [modals.md](modals.md).

1. Build sign-in, sign-out and role routing. Implement the shared profile with own name and password changes. Set up admin and teacher sidebars with their named sections and server access checks.
2. Add Classroom and StudentProfile. Build the admin screens and dialogs for teacher → class → student provisioning. Validate the role of the class teacher and create the student account/profile together.
3. Add and seed all six Lessons, ten Skills, LessonSkill scope rules and initial prerequisite links. Add the six placeholder templates extending the shared lesson layout. Use the stable identifiers and section anchors from the specs.
4. Verify direct endpoint permissions, class ownership and blank roster or curriculum states. Use a repeatable local fixture or seed command for demo accounts when helpful.

**Done when:** the admin can create a teacher, assign that teacher a class, and add a student to exactly one class. Teacher and student can sign in and update their own profile. A teacher cannot access another teacher's class, and a repeat seed leaves six lessons and ten skills.

## Phase 3 — Shared assessments and student journey

Read [user_features.md](user_features.md), [database.md](database.md), [pages.md](pages.md), and [modals.md](modals.md).

1. Add assessment forms, fixed items, class release, attempts and answers. Build admin authoring and publication for a comparable pretest and posttest covering ten skills. Keep fixtures separate from approved forms.
2. Build `assessment.html` with Start and Submit dialogs, persisted draft answers, server grading and replay-safe final submission. Keep answer keys on the server. Show the first score on submission and both after posttest.
3. Build the student journey states and server guard that opens all six lessons after pretest submission. Build teacher posttest release and its dialog. Hide or gate posttest until the class release and pretest requirements are met.

**Done when:** a new student cannot load lessons or practice before submitting the pretest, can open every lesson afterward, and can see and take the posttest only after their teacher opens it. Check that submission scores are stable on repeat requests and that teacher release is scoped to their class.

## Phase 4 — Teacher exercise authoring and publication

Read [user_features.md](user_features.md), [database.md](database.md), [pages.md](pages.md), [modals.md](modals.md), and [special_algo.md](special_algo.md).

1. Add exercises, revisions, questions, choices, and class assignments. Build the editor with title, question count, required skills, teacher pool order, hints, and difficulty. Implement text, figure upload with description, and YouTube with editable start time and timestamp extraction.
2. Build draft save/edit, publish to selected own classes, delete a never-published draft, unpublish, and discard-unsaved-changes behavior. Validate pool size and coverage before publishing and freeze the revision. Keep uploaded figures needed by old revisions.
3. Connect published exercise cards to their lesson and student class. Keep the same cards when lesson content is still a placeholder. Check invalid media, ownership and empty states.

**Done when:** a teacher can publish a valid pool to chosen classes, students in those classes see the exercise after the pretest, and unauthorized classes do not. Test all three question formats, invalid pool rejection and published version stability.

## Phase 5 — BKT, agent, adaptive practice, and decision logs

Read [database.md](database.md), [bkt.md](bkt.md), [agent.md](agent.md), [special_algo.md](special_algo.md), [r_learning.md](r_learning.md), [pages.md](pages.md), and [modals.md](modals.md).

1. Install and verify the two fitted artifacts and exact model runtime in Docker. Confirm ten BKT skill keys, checksum linkage, feature order and real positive-class probability output. Add model/version validation and record the tested pins.
2. Add session, served question, attempts, mastery, mastery event, selection log and feedback summary records. Implement a once-only first-answer transaction and repeat-safe progression. Capture the active answering duration and video time per the feature contract.
3. Implement the shared practice UI for start, resume, first answer, teacher hint, one retry, finish and one retake. Lock sessions to a published revision. Implement coverage, difficulty, score ranking, fallback and review rules. Save the 12 selected pre-answer inputs and candidate/choice data for offline evaluation.
4. Build and verify a single-skill exercise and a mixed Foundations exercise. Display first-try score and retry corrections independently. Save the deterministic feedback summary at completion.

**Done when:** the student's behavior can change the next question while required coverage and length hold. Verify exact one-skill mastery update, duplicate requests, resume, one retry, one retake with unseen preference, fallback, fixed finish, score math, feature provenance and the linked decision/outcome. An active contextual bandit is a later pilot.

## Phase 6 — Local RAG and developer content handoffs

Read [rag.md](rag.md), [file_structure.md](file_structure.md), [tech_stack.md](tech_stack.md), and [special_algo.md](special_algo.md).

1. Provide the schema/example JSON, the handoff instructions for each lesson and asset location, and stable review anchors. Integrate approved HTML and knowledge if available. Use placeholders and fallback when those deliveries are pending.
2. Implement validated skill-ID retrieval and the local llama.cpp client. Add pinned model acquisition, verification, Docker inference service and local Compose option. Persist one session result with deterministic fallback, and replace only feedback prose if generation succeeds. Keep result and mastery commits independent of inference.
3. Verify a real Qwen response with labeled example knowledge, missing/invalid knowledge fallback, failed and slow inference, duplicate generation protection, and exact review routes.

**Done when:** adaptive practice works with placeholder lessons and no JSON, a valid knowledge file produces grounded feedback from the real local model, and disabling inference leaves accurate results and a useful fallback.

## Phase 7 — Role reports and release verification

Read [user_features.md](user_features.md), [pages.md](pages.md), [modals.md](modals.md), [database.md](database.md), [agent.md](agent.md), and [tech_stack.md](tech_stack.md).

1. Complete the admin overview, teacher class and individual reports, progress labels, Chart.js summaries and accessible table equivalents. Add the required empty, loading, error and permission states. Verify all 14 main templates and 11 required dialogs.
2. Run the full flow from admin setup to posttest, including mixed practice, score comparison, feedback fallback and teacher reporting. Check keyboard flow and responsive layouts on a phone below 475 px and on desktop. Check just below and at any active breakpoint that changes navigation, dialogs, or the student journey. Verify no answer keys or other classes' results are exposed.
3. Run migrations, concurrency and replay checks on PostgreSQL in a production-like test environment. Rebuild from a clean checkout and verify persisted uploads, artifact loading and the no-npm workflow. Inspect the decision records for the retraining feature contract. Save a short verification record and list pending external content by name.

**Done when:** the complete v1 flow passes on SQLite and PostgreSQL, no required page or dialog is missing, and a teammate can reproduce the app using the README.

## Phase 8 — Deploy on Railway

Read the Railway section of [tech_stack.md](tech_stack.md), [file_structure.md](file_structure.md), and [rag.md](rag.md).

1. Prepare the GitHub revision and deployment config. Verify production settings, reproducible image builds, migrations and static asset collection. Use the owner's authorized GitHub repo and Railway project and configured access. If access is pending, prepare all deploy inputs and report the specific remaining action.
2. Deploy the Django Docker service, Railway PostgreSQL, persistent media volume, and private llama.cpp Docker service. Configure production environment, model paths, private endpoints, HTTPS host/CSRF settings and migration hooks. Create the first admin through the documented command. Load approved curriculum or use labeled demo content when appropriate.
3. Verify the public URL and health status. Smoke test sign-in, class and teacher/student setup, pretest, assigned adaptive practice, Qwen feedback with knowledge, fallback, teacher reports, posttest, and score comparison. Test figure persistence across a redeploy and inference memory and response time with several concurrent finishes.
4. Hand off the deployed URL, revision, service configuration, production runbook, backup/rollback steps and the smoke test results. Identify any approved teaching content still pending.

**Done when:** the Django and inference services are healthy, production uses PostgreSQL, required flows pass on the deployed Railway app, and the verified URL and operational instructions are handed over.
