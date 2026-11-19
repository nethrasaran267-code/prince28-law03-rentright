# RentRight

Explains tenant and landlord rights under Tamil Nadu's 2017 tenancy law and checks a rental agreement clause by clause.

**Status:** Day 1 foundation (FastAPI skeleton, quality gates). Live URL from Day 5.

## Who it's for

Third-year student from Madurai moving into a shared flat in Velachery whose landlord asks for 10 months' advance.

## The problem

Many tenants in Tamil Nadu rent houses without fully understanding their legal rights and responsibilities. Some rental agreements contain conditions that may not comply with the Tamil Nadu Tenancy Act, 2017. Tenants may find it difficult to identify unfair terms or understand complicated legal language. RentRight aims to simplify this process by explaining tenant rights and checking rental agreement clauses against applicable laws.

## Data sources

Tamil Nadu Regulation of Rights and Responsibilities of Landlords and Tenants Act 2017 and Rules, Model Tenancy Act 2021, Tamil Nadu tenancy portal guidance, registration and stamp-duty notes.

- https://www.tenancy.tn.gov.in/
- https://www.indiacode.nic.in/
- https://tnreginet.gov.in/
- Terms: Public law. Cite section numbers.

## Hardest problem (my interview story)

Act vs agreement: an uploaded agreement must be split into clauses, each checked against the right sections. This needs clause segmentation plus retrieval per clause, and a clear 'conflicts with the Act' label only when the text really conflicts.

## Signature feature

Agreement checker: upload a rental agreement and see each clause marked OK / Check / Conflicts with the Act, with the section cited.

## Target metric

Clause labels agree with a reviewer for 85%+ of clauses in 10 agreements.

## Run it locally

```bash
uv sync
cp .env.example .env   # Windows: copy .env.example .env
uv run uvicorn rentright.api.main:create_app --factory --reload
```

Then open http://localhost:8000/docs

## Checks

```bash
uv run pytest
uv run ruff check .
```


## Run locally in one command

You need Docker Desktop running. From a fresh clone:

```bash
docker compose up --build
```

This starts Postgres (with pgvector), Redis, runs the migrations and loads the sample data once, then starts the API.

- API docs: http://localhost:8000/docs
- Liveness: http://localhost:8000/health
- Readiness (checks the database): http://localhost:8000/ready

Stop with `Ctrl+C`, then `docker compose down`. Add `-v` to also delete the database volume.

## Develop and test

| Task | Mac / Linux | Windows |
|---|---|---|
| Run the API with reload | `make dev` | `uv run poe dev` |
| Unit tests (no network) | `make test` | `uv run poe test` |
| Integration tests (needs `docker compose up`) | `make test-integration` | `uv run poe test-integration` |
| Coverage report | `make cov` | `uv run poe cov` |
| Lint and format check | `make lint` | `uv run poe lint` |



## D04 · Docker, Compose and tests

### The image

```text
docker images rentright:dev
Output:
IMAGE           ID             DISK USAGE   CONTENT SIZE   EXTRA
rentright:dev   f7de2d45c2cc        300MB           70MB 

docker run --rm rentright:dev id
Output:
uid=999(app) gid=999(app) groups=999(app)
```

- Size: 300 MB (target: under 400 MB) (Content Size: 70 MB)
- First build: 314.6 seconds. After changing one line of code: 14.7 seconds.
- Which step said CACHED, and why? Docker reused previously built layers such as dependency installation and system-user setup because their inputs had not changed. The changed application code was copied again.

### The stack

```text
docker compose ps -a
Output:
NAME                  IMAGE                    COMMAND                  SERVICE   CREATED              STATUS                        PORTS
rentright-api-1       rentright-api            "sh -c 'exec uvicorn…"   api       About a minute ago   Up 52 seconds (healthy)       0.0.0.0:8000->8000/tcp, [::]:8000->8000/tcp
rentright-db-1        pgvector/pgvector:pg17   "docker-entrypoint.s…"   db        About a minute ago   Up About a minute (healthy)   0.0.0.0:5433->5432/tcp, [::]:5433->5432/tcp
rentright-migrate-1   rentright-migrate        "sh -c 'alembic upgr…"   migrate   About a minute ago   Exited (0) 52 seconds ago     
rentright-redis-1     redis:7-alpine           "docker-entrypoint.s…"   redis     About a minute ago   Up About a minute (healthy)   0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp

Explanation:
API - Healthy
DB - Healthy
Migrate - Exited
Redis - Healthy


docker compose logs migrate   (first up, then after down + up)
Output (UP):
migrate-1  | INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
migrate-1  | INFO  [alembic.runtime.migration] Will assume transactional DDL.
migrate-1  | INFO  [alembic.runtime.migration] Running upgrade  -> 9e6ea1b83dab, create documents and pages
migrate-1  | INFO  [alembic.runtime.migration] Running upgrade 9e6ea1b83dab -> 23130b270b11, add gin index on metadata
migrate-1  | records=20 inserted=20 updated=0 unchanged=0

Output (DOWN+UP): <docker compose down>
✔ Container rentright-api-1     Removed
✔ Container rentright-migrate-1 Removed  
✔ Container rentright-redis-1   Removed
✔ Container rentright-db-1      Removed 
✔ Network rentright_default     Removed
<docker compose up -d>
✔ Container rentright-redis-1   Healthy
✔ Container rentright-db-1      Healthy
✔ Container rentright-migrate-1 Exited 
✔ Container rentright-api-1     Started
Explanation:
The Docker successfully removed and recreated all containers and the network while preserving named-volume data. PostgreSQL and Redis became healthy, migrations completed successfully, and the FastAPI container started.
```

### Breaking the database on purpose

```text
db stopped:  /health → {"status":"ok","app":"rentright","env":"local"}
db stopped:  /ready  → {"status":"not_ready","checks":{"database":false}}
db started:  /ready  → {"status":"ready","checks":{"database":true}}
```

Why should a load balancer stop sending traffic here but not restart the app? (2 lines)
The load balancer should stop routing traffic because the application cannot access its database. The application itself is still alive, so restarting it is unnecessary. Once the database recovers, readiness becomes healthy and traffic can resume.

### Tests and coverage

```text
uv run poe test              → 16 passed 
uv run poe test-integration  → 7 passed
coverage TOTAL, unit only    → 73%
coverage TOTAL, with integration tests → 84%
```

Two files or lines my coverage report says are never run, and whether they matter: (2 lines)
src/rentright/store.py – 0% coverage; investigate whether this module is used by the current application flow and add tests if needed.

src/rentright/db/repository.py – initially 27% unit-only coverage; integration tests increased it to 89%. A few branches and lines remain untested.
