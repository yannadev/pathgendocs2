# Pathgen user features

See [system.md](system.md) for the curriculum and [pages.md](pages.md) for screen locations. The server checks both role and ownership for every protected action, including direct URLs, form submissions, uploads, report data, and session endpoints.

## Admin

- Sign in with a provisioned admin account. Use Django's authentication and password handling for all roles. The application admin creates teacher and student accounts. There is no public registration.
- Create an existing-teacher account first, then create a class and assign that teacher. A teacher can have several classes. Create each student with exactly one existing class. Make account and student profile creation atomic. Keep usernames unique and distinct from editable display names. Use Django password validators and hashed passwords.
- The admin/developer owns six lesson pages and their interactive content. The dashboard shows the existing lessons and skill mappings. Lesson HTML and assets come from developer handoff. Teachers add tracked exercises and questions beneath those lessons.
- Author one shared pretest and one comparable shared posttest. Match their coverage of all ten skills and align question counts and difficulty per skill. Use different items where possible. Validate the assessment form before publishing. Keep published content and answer keys stable for started attempts.
- Show an overview of accounts and class rosters. Individual mastery and score reports are a teacher/student feature by default. Expanding admin report access is an open decision.

## Teacher

- See only the teacher's assigned classes, rosters, class completion summaries, skill gaps, and individual results. Individual results include exercise completion, first-try scores and retry corrections, progress labels, and pretest/posttest scores when submitted. Show missing scores as unavailable. Compute class rates from the relevant roster and assignment, and label the denominator. Use Chart.js for charts with a text or table equivalent.
- Create a draft exercise for an existing lesson. Set title, number of questions per session, the required skill set for mixed practice, and an ordered pool. Each selected required skill gets at least one question. For a single skill lesson, that skill is implicit.
- For each question choose text only, figure plus text, or video plus text. Always enter a prompt, four answer choices, one correct choice, one primary skill, Easy/Medium/Hard, and a hint. Figure questions accept PNG or JPEG and a short meaningful image description. Video questions accept a YouTube link and editable start time. Extract a timestamp if present. Show an inline media check/preview in the editor. A full student-style preview dialog is optional.
- Publish to one or more classes assigned to that teacher. All students in those classes can start after submitting the pretest. Validate question format, pool size, required skill coverage, and class ownership on the server. Edit a draft for future sessions, with existing sessions tied to their published revision. Unpublishing stops new sessions and retakes. **V1 default:** let already-started sessions finish, with saved results preserved.
- At the end of the unit open the shared posttest for a selected class. Students who have submitted the pretest can start it when released. Student exercise completion is available as evidence to the teacher, with release controlled by the teacher.
- Change own display name and password on the shared profile page.

## Student

- Change own display name and password. The account ID links history and mastery to the same person.
- See the pretest first on the journey. Start and submit it with the assessment dialogs. **V1 defaults:** fixed item order, no hints, one final submission, all items answered before submit, and no countdown. Saved drafts can resume. Scores appear immediately after submission. Keep answer keys server-side.
- After submitting the pretest, access all six lessons and their assigned practice. Show per-skill labels such as **Not yet practiced**, **Needs practice**, and **Doing well**. Show stage completion based on assigned exercises and a distinct empty state if none are assigned. Use the prerequisite rule in [special_algo.md](special_algo.md) to offer review.
- Start an assigned exercise and see progress such as Question 2 of 5, the prompt, media where applicable, and four choices. The first submitted answer is final as BKT evidence. After a wrong answer, the student may request the hint, submit one retry, or continue. After the retry, the student can continue even if it was incorrect. Built-in lesson examples remain available for unlimited untracked practice.
- Resume an unfinished exercise at the saved question and attempt state. Offer one retake after the first session completes. Start it from current mastery and prefer questions the student has not seen. Keep both results available. A retake is a separate session.
- Show the first-try score, additional answers corrected after retry, and short feedback on one next focus. Show a specific lesson section or eligible assigned review exercise when available. The student chooses where to go afterward.
- See **Waiting for your teacher** at the posttest stage before release. After submission, show the two scores side by side. Calculate score change in percentage points when the forms have different item totals, and label it as score change.

## Permissions and data

- Students may read only their own results and assigned practice. Teachers may read results for their own class and change their own exercises, assignments, and class posttest releases. Application admins manage accounts, classes, curriculum metadata and shared assessment content. Use Django's internal management tooling separately from the product navigation.
- Apply CSRF protection to writes, enforce permissions on the server, and return suitable forbidden or not found responses. Keep answer keys, teacher-only data, and other students' results out of student HTML and JSON. Use POST for sign-out and other writes.
- Draft exercise deletion is available only before the exercise has published history. Use unpublish to withdraw assigned practice. The optional account deactivation and student transfer UI are documented in [modals.md](modals.md).
