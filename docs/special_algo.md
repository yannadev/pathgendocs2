# Required adaptive rules for Pathgen v1

These rules turn BKT estimates and predicted struggle into practice decisions. The teacher's published pool and fixed exercise length define the boundaries.

## 1. Validate the pool and protect required skills

Before publishing, require a positive session count, enough distinct valid items, and at least one item for each teacher-selected required skill. The number of required skills cannot exceed the session count. Single-skill exercises require that skill. Each question has one allowed primary skill and four choices. Validate selected classes and freeze the published revision.

For each selection, start with unused questions from the session's revision. A candidate is eligible if taking it still leaves enough remaining slots and available questions to cover all required skills not yet encountered. Count skill coverage when a question is completed, regardless of correctness. Never serve the same question twice in a session. If a corrupted revision prevents any legal choice, keep saved progress and show a recoverable error to the student and teacher.

## 2. Pick a skill and a manageable challenge

Use the following deterministic v1 defaults, with named settings and a recorded policy version:

1. For a first-ever tracked question without usable history, pick the first eligible item in teacher pool order.
2. If the remaining slots are all needed for required coverage, choose among uncovered required skills. Otherwise consider the eligible skills. For a mixed exercise choose the skill with the lowest current BKT mastery among those options, breaking ties by the earliest eligible teacher pool item. A single-skill exercise always uses its own skill.
3. In that skill, prefer eligible items the student has not seen in the original session when selecting for a retake. Compare stable item keys across revisions. If no unseen candidate in the skill remains, allow a previously seen item once in the retake. Coverage takes priority.
4. If the student's last two completed encounters on this skill were struggles, prefer easier candidates when available. If both were independent successes, prefer harder candidates when available. Use the difficulty of the most recent encounter as the reference. Use all candidates in the skill when neither pattern applies. If a preferred level is absent, keep the closest eligible difficulty to the desired direction.
5. Rank candidates in the remaining set by distance to a provisional target struggle probability of 0.30. On ties use teacher pool order and stable item ID. The target is a practice setting for pilot validation. If usable history, mapping or model output is missing, use teacher order within the same coverage, skill, and available difficulty preferences. Log the fallback and candidate set.

A struggle is an incorrect first answer, a retry, or hint use on a completed encounter. The first-try score and mastery observation are separate from the choice of difficulty. For a chosen deterministic action, the selection probability is 1 even when its predicted struggle is 0.30.

## 3. Grade once and finish at the set length

Grade and persist the first submission, apply one BKT update, and display its result. After a wrong first try offer the hint and one retry, or let the student move on. Persist one decision per served item. Resume the saved position and reuse prior results on repeated requests. On the last completed item finalize the session once. Show the first-try score and number corrected on retry. High mastery can increase challenge while the session continues to its set length. Allow one retake after the first completed session.

## 4. Report progress and suggest review

**V1 label defaults:** no tracked evidence → **Not yet practiced**. With evidence, BKT mastery below 0.75 → **Needs practice**. At or above 0.75 → **Doing well**. These are display thresholds, not assessment grades or access rules. Use Pathgen prerequisite links to suggest review after pretest unlock. If a prerequisite has no evidence, use the not-yet-practiced wording.

After exercise completion, prioritize a practiced skill with the most struggle encounters in that session, then lower mastery, then curriculum order. If there was no struggle, suggest continued practice on the lowest-mastery skill seen in the session. Report a strength only when supported by an independent correct answer. Resolve a valid review destination through the curriculum. Prefer the exact section when it exists, then a relevant available lesson or assigned review exercise. Missing delivered content can leave the link absent. Feedback uses [rag.md](rag.md).

Acceptance cases: five slots covering four required skills; insufficient-pool publish rejection; repeated subtraction errors with an easier option and with no easier option; repeated independent success; deterministic fallback; no early finish; and a retake that prefers unseen items without losing coverage.
