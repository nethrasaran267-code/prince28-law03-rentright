# Engineering notes: RentRight

## D03 · PostgreSQL

### Migrations (upgrade, downgrade, upgrade)

```text
paste the output of:
uv run alembic downgrade -1
uv run alembic current
uv run alembic upgrade head
uv run alembic history
```

### The loader is idempotent

```text
run 1: records=20 inserted=20 updated=0 unchanged=0
run 2: records=20 inserted=0 updated=0 unchanged=20
run 3 (after editing one record): records=20 inserted=0 updated=1 unchanged=19
```

### EXPLAIN for ?section=VALUE

(rentright) PS C:\Users\Ganga Devi\OneDrive\Documents\Projects\prince28-law03-rentright>uv run python scripts/explain.py section "Rental Agreement"
-- 1) What the planner chooses on its own:
Seq Scan on documents  (cost=0.00..11.00 rows=1 width=36) (actual time=0.021..0.021 rows=0.00 loops=1)
  Filter: (metadata @> '{"section": "Rental Agreement"}'::jsonb)
  Rows Removed by Filter: 20
  Buffers: shared hit=1
Planning:
  Buffers: shared hit=149
Planning Time: 0.389 ms
Execution Time: 0.055 ms

-- 2) Seq scan switched off, to prove the index can be used:
Bitmap Heap Scan on documents  (cost=12.81..16.82 rows=1 width=36) (actual time=0.033..0.033 rows=0.00 loops=1)
  Recheck Cond: (metadata @> '{"section": "Rental Agreement"}'::jsonb)
  Buffers: shared hit=3
  ->  Bitmap Index Scan on ix_documents_metadata_gin  (cost=0.00..12.81 rows=1 width=0) (actual time=0.010..0.010 rows=0.00 loops=1)
        Index Cond: (metadata @> '{"section": "Rental Agreement"}'::jsonb)
        Index Searches: 1
        Buffers: shared hit=3
Planning:
  Buffers: shared hit=1
Planning Time: 0.079 ms
Execution Time: 0.087 ms

Why did the planner choose a Seq Scan on my table? (2 lines, your own words)
Sequential Scan is a method that checks all the 20 rows of a table to check for a given record instead of using an index. Because checking all the rows is comparatively cheaper.

When would it switch to the GIN index? (1 line)
GIN stands for Generalized Inverted Index.
PostgreSQL may switch to using a GIN when the table grows (becomes a large data set) and the query planner estimates that searching through the index is more efficient than scanning the entire table.



It is a method that stores mappings between individual values in complex data types and the rows containing them, allowing PostgreSQL to find matching records efficiently instead of scanning the entire table. This saves time and cost in case of laarge data sets (compared to Seq Scan).



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



## D05 · CI/CD and going live

- Live URL: https://rentright-r7vs.onrender.com
- CI run for pull request 1: https://github.com/nethrasaran267-code/prince28-law03-rentright/actions/runs/44782167418
- First deploy that ran by itself: https://github.com/nethrasaran267-code/prince28-law03-rentright/actions/runs/45112627771
- Merge clicked at 09:49 AM → "Smoke test passed" at 09:50 AM → lead time: 01 min


### Smoke test output from that CD run

```text
waiting: https://rentright-r7vs.onrender.com/health -> None
waiting: https://rentright-r7vs.onrender.com/health -> 200 9e14611
ok  /health -> 200, version 5780ef3c379917981f5f4402487ef8b13049f052
ok  /ready -> 200 {'status': 'ready', 'checks': {'database': True}}
ok  /v1/documents?limit=1 -> 200
Smoke test passed
```

### Rollback drill

- Rolled back to sha 40ba8fddcabe28976433ee29acb35f7a67256e3d : started 10:11 AM, smoke passed 10:13 AM (2 min)
- Rolled forward to sha 5780ef3c379917981f5f4402487ef81b3049f052 : 5 min
- During the rollback, the database stayed on the newer schema. Why is that safe here? 
The CD workflow skips database migrations during manual rollback, so the database is not downgraded.
This is safe when the newer schema is backward-compatible with the older application version, allowing it to continue working without requiring a schema downgrade.

### Where my secrets live

- Neon string: GitHub Secret DATABASE_URL + Render environment. Not in Git, not in the image.
- Render deploy hook: GitHub Secret RENDER_DEPLOY_HOOK only.

### Monitoring

- UptimeRobot checks /health every 5 minutes; alerts go to my email.
- One thing that would make my live app faster on its first request: (1 line)
