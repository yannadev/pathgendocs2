# Pathgen v1 system specification

Pathgen teaches Grade 7 integer mathematics through six lessons, teacher-authored adaptive practice, and a shared pretest and posttest. This specification is the implementation contract for the repository. A builder can use it without earlier chats.

## How to use the specification

- Start with [build_order.md](build_order.md). When asked to execute a phase, read this file and the documents named by that phase, check its prerequisites in the repository, implement its tasks, and run its acceptance checks.
- The files below own their respective topics. Use their detailed rules when this overview summarizes a topic. If a genuine conflict blocks implementation, identify the conflicting requirements and continue with unaffected work.
- These 13 named files are the active specification. Numbered documents from earlier drafts, if present elsewhere, are historical. Test fixtures and examples do not establish product requirements.
- Treat a marked **v1 default** as a reversible design choice. Treat an **open decision** as a choice to record and resolve if a later change depends on it. Only implement features in the requested phase.

| File | Owns |
|---|---|
| [user_features.md](user_features.md) | Roles, permissions, classroom workflows, student experience |
| [pages.md](pages.md) | The 14 main templates, routes, navigation, and screen states |
| [modals.md](modals.md) | The 11 required dialogs and 7 optional dialogs |
| [tech_stack.md](tech_stack.md) | Required stack, Docker handoff, environments, Railway topology |
| [file_structure.md](file_structure.md) | App boundaries, repository layout, and artifact destinations |
| [database.md](database.md) | Model ownership, fields, constraints, and data lifecycle |
| [bkt.md](bkt.md) | Bayesian Knowledge Tracing parameters, math, and evidence rules |
| [agent.md](agent.md) | Trained model interface, exact features, evaluation, retraining data |
| [special_algo.md](special_algo.md) | Required question selection and progression rules |
| [r_learning.md](r_learning.md) | Decision logging in v1 and the later contextual bandit experiment |
| [rag.md](rag.md) | Developer JSON handoff, retrieval, Qwen, and feedback fallback |
| [build_order.md](build_order.md) | Ordered implementation phases and acceptance checks through Railway deployment |

## How a student moves through Pathgen

1. The admin creates a teacher, creates a class with that teacher, then creates students and assigns each to exactly one class.
2. The student signs in and takes the admin's fixed pretest. Submission unlocks all six lessons. The pretest measures prior knowledge. **V1 default:** practice mastery starts from the fitted BKT prior for each skill.
3. The dashboard shows a journey map from pretest through the lessons to posttest. Lesson order is suggested. Students can revisit lessons and repeat their built-in activities. Built-in activities do not enter tracked practice history or BKT.
4. The student starts a teacher-published exercise assigned to their class. One shared exercise screen serves every lesson and question format. The teacher sets the question count. Each question has four answer choices and one primary skill.
5. The first answer gives BKT one observation. A wrong first answer enables a teacher hint and one retry. At question completion the agent scores eligible candidates and the fixed selection rules select the next question. One retake is available after a completed first session.
6. At the teacher's question count, show the first-try score, any additional answers corrected on retry, a strength if the evidence supports one, and one focus skill with a review link when available. Local RAG writes brief feedback after completion.
7. The teacher opens the shared posttest for their class. After submitting, the student sees both assessment scores. The teacher sees completion, skill gaps, and assessment results for their own class and students.

## Curriculum contract

Seed stable codes and route names. Use these skill codes in the database, model features, published question versions, and knowledge JSON.

| Lesson code | Template filename | Skills taught |
|---|---|---|
| `integer_foundation` | `integer_foundation.html` | 1 `INT_NUMBER_LINE` Identify and represent integers and zero on a number line. 2 `INT_OPPOSITE_POS4` Find the opposite. 3 `INT_ABS_COMPARE` Interpret and calculate absolute value. 4 `INT_ORDER_NEG` Compare and order integers. |
| `integer_addition` | `integer_addition.html` | 5 `INT_ADD_NEG` Add integers. |
| `integer_subtraction` | `integer_subtraction.html` | 6 `INT_SUB_NEG` Subtract integers. |
| `integer_multiplication` | `integer_multiplication.html` | 7 `INT_MULT_NEG` Multiply integers. |
| `integer_division` | `integer_division.html` | 8 `INT_DIV_NEG` Divide integers. |
| `integer_expressions_applications` | `integer_expressions_applications.html` | 9 `INT_EXPR_NEG` Evaluate integer expressions using order of operations. 10 `INT_CONTEXT` Solve situations involving signed quantities and changes. Explicit cumulative review can also use skills 5–8. |

The historical model codes identify Junyi mappings. Some mapped Junyi tasks represent only part of the wider Pathgen objective, especially number line, opposites, and absolute value. Teach and assess the wider objective using the local curriculum. Prerequisites are review suggestions based on Pathgen's curriculum, with subtraction drawing on opposites and addition as the initial example.

## Runtime and scope

| Component | Responsibility |
|---|---|
| BKT | Update one student's mastery for the primary skill on the first submitted answer. |
| Supervised agent | Estimate struggle risk for a candidate using only earlier behavior and the candidate's known attributes. |
| Fixed selection rules | Choose an eligible unused question, preserve required coverage, adjust challenge, and stop at the fixed count. |
| RAG | Retrieve approved developer-authored knowledge and generate a short result summary locally. The server owns score, focus skill, and review URL. |
| Contextual bandit | Future pilot. V1 records decisions and outcomes. The rules govern live choices throughout v1. |

Developer handoffs can arrive independently. Install placeholder lesson templates with working exercise lists, and ship a sample knowledge schema. Practice, assessment, and reporting must work during both handoffs. See [rag.md](rag.md) for the feedback fallback and [file_structure.md](file_structure.md) for installation paths.

Open decisions: whether application admins should have a view of individual progress, whether pretest responses should later initialize BKT, and whether the professor requires embedding similarity in addition to structured retrieval. For v1, application admins have account and class administration, teachers see their own class results, BKT uses fitted priors, and RAG retrieves by skill ID. Add a different behavior only after that choice is made.
