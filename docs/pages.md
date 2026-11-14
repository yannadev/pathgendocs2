# Pathgen pages and navigation

The 14 main page templates below are rendered by Django. Template names are distinct from public routes. Shared layouts, section partials, dialogs and different states of one screen are additional template files, with no separate public page required.

| # | Template | Proposed route | Access and content |
|---|---|---|---|
| 1 | `accounts/login.html` | `/login/` | Shared sign-in with role-based landing. Show invalid credentials in the form. |
| 2 | `accounts/profile.html` | `/profile/` | Own display name and password forms for teachers and students. Reuse for admin profile if needed. |
| 3 | `dashboards/admin_dashboard.html` | `/admin-console/<section>/` | Overview, accounts, classes, lessons, and assessments. Include account/class creation and shared assessment editing. Keep Django's internal admin route distinct. |
| 4 | `dashboards/teacher_dashboard.html` | `/teacher/<section>/` | Overview, My Classes, and Exercises. Class and student drilldowns use this template. Link to the editor. |
| 5 | `practice/exercise_editor.html` | `/teacher/exercises/new/`, `/teacher/exercises/<id>/edit/` | Lesson, title, count, required skills, ordered pool, format and media fields, draft actions, and publishing to classes. |
| 6 | `dashboards/student_dashboard.html` | `/learn/` | Journey map, current action, skills, and pretest/posttest status and scores. |
| 7 | `lessons/integer_foundation.html` | `/learn/lessons/integer_foundation/` | Foundations, skills 1–4. |
| 8 | `lessons/integer_addition.html` | `/learn/lessons/integer_addition/` | Addition, skill 5. |
| 9 | `lessons/integer_subtraction.html` | `/learn/lessons/integer_subtraction/` | Subtraction, skill 6. |
| 10 | `lessons/integer_multiplication.html` | `/learn/lessons/integer_multiplication/` | Multiplication, skill 7. |
| 11 | `lessons/integer_division.html` | `/learn/lessons/integer_division/` | Division, skill 8. |
| 12 | `lessons/integer_expressions_applications.html` | `/learn/lessons/integer_expressions_applications/` | Expressions and Applications, skills 9–10. May list cumulative operation review. |
| 13 | `practice/exercise.html` | `/learn/exercises/<id>/` | Shared start, resume, question, retry, retake, and results states. Optional session-specific routes can reuse this template. |
| 14 | `assessments/assessment.html` | `/learn/assessments/<kind>/` | Shared pretest and posttest instructions, questions, submit, score, and comparison. |

Route spellings are implementation defaults. Preserve lesson codes and stable named URLs after the content handoff. Link to routes through Django URL reversing. Separate GET rendering from POST mutations, and retain the same permission checks for any supporting JSON endpoints.

## Navigation

- **Admin sidebar:** Overview, Accounts, Classes, Lessons, Assessments. **Sign out** at the bottom.
- **Teacher sidebar:** Overview, My Classes, Exercises, Profile. **Sign out** at the bottom. My Classes opens a class, then an individual student, within the teacher dashboard. Exercises opens the draft and published lists and links to the editor.
- **Student:** Journey, own profile, and sign out. In lessons, exercises and assessments, include a clear way back to the journey or originating lesson.

## Student journey

Use a stage map inspired by Candy Crush, with a clear reading and keyboard order on desktop and mobile. Order: **Pretest → Integer Foundations → Addition → Subtraction → Multiplication → Division → Expressions and Applications → Posttest**. Each lesson stage leads to a teaching page with built-in practice and its assigned exercises.

| State | What the student can do |
|---|---|
| Pretest required | Start or resume the pretest. Show why lessons are locked. |
| Lessons available | Open any lesson, see the ten skill labels, and start/resume assigned practice. Show completion counts and no-assignment states. |
| Posttest open | Start or resume the posttest after pretest submission. Keep access to the lessons. |
| Unit complete | See both scores and review or revisit lessons and practice when still assigned. |

Lesson pages extend the shared lesson layout and retain its assignment list and review targets. Placeholders show a content status and working assignment cards. Delivered HTML and scoped lesson assets can replace the teaching body independently. Show hints inline only after a wrong first answer. Put the exercise result and RAG feedback in the result state, with a score visible even while feedback is pending.

Use readable labels, semantic controls, visible keyboard focus, reduced motion for animated elements, mobile layouts for charts and lesson stages, and status messages that work without color. Give charts a text or table alternative. See [modals.md](modals.md) for overlay behavior.

Build the student journey, sidebars, forms, dialogs, charts, and lesson layouts for narrow screens first. Use the shared [mobile-first CSS breakpoints](tech_stack.md#mobile-first-css) wherever a wider layout needs an override.
