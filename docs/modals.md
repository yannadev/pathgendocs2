# Pathgen dialogs

The approved plan has **11 required v1 dialogs** and **7 optional dialogs**. Counts refer to distinct flows, even where templates or a dialog shell are reused.

## Required for v1

| # | Dialog | Page and behavior |
|---|---|---|
| 1 | Create teacher account | Admin dashboard. Create a teacher using Django account validation. |
| 2 | Create class | Admin dashboard. Enter class name and assign an existing teacher. If none exist, show the next step on the page. |
| 3 | Create student account | Admin dashboard. Enter account details and select exactly one class. |
| 4 | Publish exercise | Editor. Review the classes, question count and required skills. Resolve validation errors in the editor before confirmation. |
| 5 | Delete draft exercise | Editor or teacher exercise list. Name the exercise and explain that its draft questions and hints will be removed. Use only for a never-published draft. |
| 6 | Unpublish exercise | Editor or teacher exercise list. List affected classes. Stop new starts and retakes while allowing already-started sessions to finish and preserving saved results. |
| 7 | Open posttest | Teacher dashboard. Name the class and confirm its eligible students will be able to start. |
| 8 | Start assessment | Shared pretest/posttest dialog. Explain the purpose, no hints, final submission, and the pretest unlock when applicable. |
| 9 | Submit assessment | Shared pretest/posttest dialog. Confirm final submission. If an item is missing, guide the student to the first unanswered item before opening it. |
| 10 | Start your one retake | Exercise results. Show the item count and that it starts the second and final session. |
| 11 | Discard unsaved changes | Editor and other editable forms. Reuse when navigation, closing a form, or signing out would discard unsaved work. |

## Optional

| # | Dialog | Purpose |
|---|---|---|
| 12 | Preview as student | Teacher tests the item, figure/video, choices and hint in student format. |
| 13 | Preview assessment | Admin reviews the form in student format before release. |
| 14 | Enlarge figure | Student inspects a diagram and returns to the same question state. |
| 15 | Compare exercise attempts | Student compares first-try scores and retry corrections side by side. |
| 16 | Deactivate account | Admin reviews which account loses access if deactivation is added. Retain historical results. |
| 17 | Transfer student | Admin verifies the current and destination classes if transfer is added. Preserve the student identity and history. |
| 18 | Close posttest | Teacher reviews the impact on unstarted and in-progress attempts if closing is added. |

## Shared interaction

Use specific action names such as **Publish exercise**, **Keep published**, **Discard changes**, and **Keep editing**. Keep focus within an open dialog, give it an accessible name, offer a visible cancel/close control, and restore focus to the opener. Preserve submitted form values on validation errors. Block duplicate clicks while a request is pending and enforce write rules on the server.

Start regular practice directly after showing its count and retry rules. Routine sign-out is a direct action, with the discard dialog available for unsaved changes. Hints, retry controls, results, feedback and progress live on their main pages.
