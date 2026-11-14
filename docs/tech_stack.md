# Pathgen stack and deployment

## Required stack

| Layer | Technology |
|---|---|
| Application | Django with server-rendered HTML templates, plain CSS and JavaScript |
| Charts | Chart.js browser distribution |
| Icons | AnimateIcons Lucide at `@animateicons/vanilla/dist/cdn/lucide.js` |
| Fonts | Google Fonts with suitable CSS fallback fonts |
| Local development | Docker Compose with SQLite |
| Final deployment | Docker on Railway with PostgreSQL |
| Local generation | llama.cpp running Qwen GGUF, with Django reading developer-supplied JSON |

The project builds without npm, npx, Node tooling or a frontend compilation step. Load pinned Chart.js and the specified AnimateIcons Lucide browser bundle from their CDN or checked-in browser distributions. The official AnimateIcons path is `https://cdn.jsdelivr.net/npm/@animateicons/vanilla/dist/cdn/lucide.js`. Its URL contains the registry route. The browser loads it directly. Use Google Fonts through CSS and style the app with project CSS. YouTube is the external dependency for video items.

## Mobile-first CSS

Use the following predefined breakpoints for viewport-based layout changes across Pathgen, including developer-delivered lesson styles. Put the base mobile styles outside media queries so they apply below 475 px and carry forward to wider screens. Use fluid sizing and wrapping first. Add a `min-width` override only where a component needs a layout change, and order any overrides from smallest to largest. A component can use just one or two breakpoints. Leave unused queries commented or omit them.

```css
/* xs */
/* @media (min-width: 475px) {} */

/* sm */
/* @media (min-width: 640px) {} */

/* md */
/* @media (min-width: 768px) {} */

/* lg */
/* @media (min-width: 1024px) {} */

/* xl */
/* @media (min-width: 1280px) {} */

/* 2xl */
/* @media (min-width: 1536px) {} */
```

## Python runtime

Default to Python 3.14.7 and the latest patched Django 5.2.x compatible with it, with the chosen patch pinned. The existing fitted notebook records scikit-learn 1.9.1, pandas 3.0.6, NumPy 2.5.3. The owner's installed companion versions were SciPy 1.18.1, joblib 1.6.0 and threadpoolctl 3.7.0. Validate the exact pins and load the approved pickle in the Docker Linux environment in Phase 5. Keep a reproducible lock in `requirements.txt`. Use a supported PostgreSQL driver such as psycopg, Pillow for image validation, Gunicorn for production WSGI, and WhiteNoise for packaged static assets.

## Teammate setup

1. Clone the repository, install Docker with Compose, and copy `.env.example` to `.env`.
2. Run `docker compose up --build` and open `http://localhost:8000`. The development startup migrates SQLite, with database and uploads in named volumes. Support an idempotent curriculum seed and an explicit admin bootstrap command. Use clearly marked demo content if approved teaching material is pending.
3. Provide a documented Compose option or profile for the local inference service with a cached verified GGUF. Core flows and feedback fallback also work with that profile off. Document artifact placement and availability checks.
4. Verify a teammate or clean checkout can run the same commands with only documented prerequisites. Keep passwords, real learner data, local databases, media and GGUF binaries out of Git.

## Railway production

- Deploy the Django Dockerfile and PostgreSQL service in the same Railway environment. Use a private connection reference for `DATABASE_URL`. Require production secrets and explicit allowed hosts and trusted HTTPS origins. Disable debug and run Django's deployment checks.
- Run database migrations in a Railway pre-deploy command using PostgreSQL. Keep collected static assets in the web image. Bind Gunicorn to Railway's `PORT`, configure health checks, and collect static files for WhiteNoise.
- Mount a persistent volume for teacher-uploaded figures at the configured media path. Serve them through authorized Django routes or an equivalent scoped implementation. Keep volume writes at runtime. Confirm an uploaded figure survives a redeploy. Document storage and database backup procedures.
- **Deployment default:** run a separate private llama.cpp Docker service in the same Railway environment. Supply the verified model in the inference image or provision it into persistent runtime storage. Keep its endpoint private and set Django's `LLM_BASE_URL` to the private hostname and port. Check readiness, model memory, concurrent latency and fallback on the selected Railway plan.
- Keep SQLite and PostgreSQL behavior aligned through migrations and tests. Use PostgreSQL for concurrency and production smoke checks. Avoid destructive reset commands on production data.

[Chart.js browser builds](https://www.chartjs.org/docs/latest/getting-started/installation) · [AnimateIcons Plain JS](https://animateicons.in/icons/docs/vanilla#cdn) · [Google Fonts](https://developers.google.com/fonts/docs/getting_started) · [Python 3.14 compatibility in Django 5.2](https://docs.djangoproject.com/en/5.2/releases/5.2.8/) · [Django deployment checklist](https://docs.djangoproject.com/en/5.2/howto/deployment/checklist/) · [Railway Dockerfiles](https://docs.railway.com/builds/dockerfiles) · [Railway pre-deploy](https://docs.railway.com/deployments/pre-deploy-command) · [Railway volumes](https://docs.railway.com/volumes) · [Railway private networking](https://docs.railway.com/networking/private-networking)
