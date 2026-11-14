# Pathgen repository structure

Use one Django repository. Place the active specification and its entry point inside that repository so any teammate or new Codex chat has the same instructions.

```text
pathgen/
  AGENTS.md                  # Direct builders to docs/system.md and docs/build_order.md
  README.md                  # Clone, environment, Compose, fixtures, verification
  .env.example
  .gitignore
  .dockerignore
  manage.py
  requirements.txt          # Pinned Python dependencies
  Dockerfile                # Shared Django image for development and Railway
  compose.yaml              # Local SQLite, media and local model cache
  railway.toml              # Django service deployment configuration
  config/
    urls.py
    wsgi.py
    settings/{base,development,production}.py
  apps/
    accounts/               # User, sign-in and profile
    classrooms/             # Classroom, StudentProfile, account provisioning views
    curriculum/             # Lesson, Skill, LessonSkill, SkillPrerequisite, seed data
    practice/               # Exercise revisions, assignments, sessions, answers and logs
    assessments/            # Forms, releases, attempts and assessment answers
    dashboards/             # Role views and scoped reports
  learning/
    bkt.py
    agent.py
    features.py
    selection.py
    progress.py
    feedback/{retrieval,llm,service}.py
  templates/
    layouts/{base,dashboard,lesson}.html
    includes/{admin_sidebar,teacher_sidebar}.html
    includes/dialogs/
    accounts/ dashboards/ lessons/ practice/ assessments/
  static/
    css/
    js/
    images/
    lessons/<lesson_code>/  # Delivered assets organized by lesson
  content/
    lesson_knowledge.example.json
    lesson_knowledge.json   # Approved file, added when delivered
  model_artifacts/
    bkt_parameters.json
    agent_model.pkl
  deploy/
    llm/Dockerfile          # Pinned llama.cpp and Qwen service for deployment
  scripts/
    fetch_llm.py            # Download and verify Qwen into the local model cache
  docs/
    [13 active specification files]
  tests/
```

Add standard Django app modules and migrations where needed. Put Django writes and transaction coordination in the owning app. Keep pure BKT, feature building and selection logic in `learning/`. Use [pages.md](pages.md) for the exact main template filenames. Build the required dialogs in the phase that owns the action.

## Developer content handoff

- Lesson developers supply the teaching body for the matching template under `templates/lessons/` and any assets under `static/lessons/<lesson_code>/`. Give them the shared layout's content block and stable skill section anchors. Default anchor convention: `skill-` plus the lowercase skill code with underscores replaced by hyphens. Keep the generated assignment list in the shared layout. Test that the handoff preserves access guards, navigation, and review links.
- Use the approved knowledge file at `content/lesson_knowledge.json`. The separate authoring team follows [rag.md](rag.md). The sample is a schema and demo reference. A missing approved file invokes the fallback.
- Acquire the two fitted model artifacts through the owner-approved handoff. See [bkt.md](bkt.md) and [agent.md](agent.md) for known source and checksums. Local absolute paths in those documents are acquisition hints for the current owner. The app resolves files relative to its project configuration.
- Keep the Qwen binary, uploaded media, SQLite files, logs, credentials, and real learner data out of Git. A model cache supplies Qwen locally. Production ships or provisions the verified Qwen binary for the private inference service. Approved lesson knowledge and approved small fitted artifacts can be delivered with the repo. Follow [tech_stack.md](tech_stack.md) for deployment.
