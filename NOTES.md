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
- One thing that would make my live app faster on its first request: Preload frequently used data and initialize models during application startup instead of loading them on the first user request.

## D06 · Collecting the corpus

- **Start target:** TN Act + Rules + Model Tenancy Act.
- **Collected today (DATASET.md):** 10 files, 266 pages, 7.0 MB.
- **Doors used:** HTTP downloads and existing local files.

### Licence check

| Source | Licence or terms, in my words | May I show full text in answers? |
|---|---|---|
| PRS India | PRS-hosted legal text; verify against official Gazette | Not confirmed |
| Tamil Nadu Gazette | Government Gazette; verify reuse terms | Not confirmed |
| Tamil Nadu Housing Board | Public legal text; verify reuse terms | Not confirmed |
| TN Registration | Government publication; verify reuse terms | Not confirmed |
| MoHUA | Government-hosted Model Tenancy Act; reuse terms need verification | Not confirmed |

### robots.txt

The downloader checks robots.txt before requesting source documents. Three URLs are recorded as blocked by robots.txt in the cumulative manifest. Unreachable robots.txt files are treated as disallowed.

### Download results

**Cumulative manifest (21 unique URLs):**
- Downloaded: 10
- Blocked: 3
- Failed: 8

**Earlier Run 1 (23 November):**
- Sources: 10
- Downloaded: 0
- Skipped: 0
- Blocked: 2
- Failed: 8
- Duration: 4.3 seconds

**Later runs:**
- A subsequent run was interrupted using Ctrl+C.
- Exact interruption count is not recoverable from the cumulative manifest.
- Run 2 and Run 3 individual summaries remain unverified.

### Capacity

- Current measured corpus: 10 files, 266 pages, 7.0 MB.
- Full corpus target: 1,000+ pages.
- Estimated full-corpus tokens: 0.65 million.
- Estimated embedding cost: $0.11 (approximately ₹11).
- Estimated vector storage: 6 MB.
- Estimated text storage: 3 MB.
- Estimated combined storage: 9 MB.

### What surprised you

The downloader could collect documents from multiple sources, but some government websites blocked access through HTTP 403 or robots.txt. Existing local files also helped complete the corpus.

### What went wrong and how you fixed it

Some source URLs were blocked or failed with HTTP 403. I used alternative available sources, including Tamil Nadu Gazette and Housing Board documents. Run 1 was also interrupted with Ctrl+C, so I used the cumulative manifest instead of guessing its interruption count.

### Known discrepancies

- `sources.csv` contains 22 source entries, while the manifest contains 21 unique URL records.
- The 2020 Gazette Rules Amendment has no corresponding manifest record.
- An earlier filesystem inspection found 15 PDFs, while DATASET.md reports 10 files. These counts need reconciliation.

## D07 · Parser comparison

<!-- parsers:start -->
| File | Parsers (A / B) | Pages | A found | B found | Time (A / B) | Better, why |
|---|---|---|---|---|---|---|
| 1960tn18-c8ca78f2.pdf | PyMuPDF / pypdf | 62 | 124,926 chars, 658 headings, 0 tables | 121,147 chars, 0 headings, 0 tables | 17.14 s / 2.15 s | |
| tnreginet-gov-779857d0.html | trafilatura / strip tags | 1 | 654 chars, 4 headings, 0 tables | 21,400 chars, 0 headings, 0 tables | 0.16 s / 0.03 s | |
| 79act-2019-c99673af.pdf | PyMuPDF / pypdf | 12 | 28,122 chars, 50 headings, 2 tables | 27,742 chars, 0 headings, 0 tables | 0.35 s / 0.26 s | |
| extra-ordinary-lists-9a776d2e.html | trafilatura / strip tags | 1 | 280,322 chars, 0 headings, 1 tables | 268,567 chars, 0 headings, 0 tables | 0.44 s / 0.03 s | |
| 413_Ex_IV_2.pdf | PyMuPDF / pypdf | 2 | 3,274 chars, 13 headings, 0 tables | 3,198 chars, 0 headings, 0 tables | 0.08 s / 0.03 s | |
| 79-ex-iii-1a-c4b0cc5b.pdf | PyMuPDF / pypdf | 12 | 28,164 chars, 50 headings, 2 tables | 27,677 chars, 0 headings, 0 tables | 0.35 s / 0.20 s | |
| tnrrrltact2017-5bbfc4bc.pdf | PyMuPDF / pypdf | 40 | 84,474 chars, 68 headings, 0 tables | 83,548 chars, 0 headings, 0 tables | 0.17 s / 0.17 s | |
| 261-ex-iv-2-7d92cfbc.pdf | PyMuPDF / pypdf | 40 | 84,474 chars, 68 headings, 0 tables | 83,548 chars, 0 headings, 0 tables | 0.22 s / 0.11 s | |
| 297_Ex_III_1a.pdf | PyMuPDF / pypdf | 1 | 1,857 chars, 15 headings, 0 tables | 1,789 chars, 0 headings, 0 tables | 0.06 s / 0.01 s | |
| 311_Ex_IV_2.pdf | PyMuPDF / pypdf | 3 | 5,197 chars, 17 headings, 0 tables | 5,074 chars, 0 headings, 0 tables | 0.03 s / 0.03 s | |
<!-- parsers:end -->

### D07 · Parsing my corpus

- Files processed in Run 1: **10**
- Newly parsed: **9**
- Already parsed: **1**
- Retry output: **10 already parsed**
- Pages reported in PostgreSQL by the initial parsing run: **134**
- Parser failures: **Not explicitly reported in the available output**

**Coverage clarification:** All 10 files in the parsing run were processed. The cumulative download manifest contains 21 unique URL records, of which 10 were downloaded. Therefore, 10/10 processing coverage applies to the 10-file parsing input, not necessarily the entire source manifest.

### Failures by reason

| Reason | Files | What I did |
|---|---|---|
| Confirmed parser failures | Not reported | No confirmed parser failure count is available. |
| Previously failed files on retry | 0 newly retried | The retry output reported that all 10 files were already parsed. |

- **The one I fixed:** No parser code fix has been confirmed. I identified and documented a PDF text-extraction issue.

### What I saw when I read my parsed pages

- A page where the heading path is right: Tamil Nadu Government Gazette, page 12, `["EXTRAORDINARY", "TAMIL NADU GOVERNMENT GAZETTE EXTRAORDINARY", "DETAILS OF APPEAL:"]`.
- Something that came out wrong: HOUSING AND URBAN DEVELOPMENT DEPARTMENT, pages 8 and 11. Examples include `fi led` instead of `filed` and `eff ected` instead of `effected`.
- A table: PyMuPDF detected 2 tables in `79act-2019-c99673af.pdf`, while pypdf detected 0. The extracted table structure and Markdown grid quality have not yet been visually verified.

### My parser choice

- For PDFs, I chose **PyMuPDF** because it detected headings and tables that pypdf did not detect in the comparison.
- For example, in `79act-2019-c99673af.pdf`, PyMuPDF detected 50 headings and 2 tables, while pypdf detected 0 headings and 0 tables.
- However, PyMuPDF was slower on the 62-page PDF: 17.14 seconds compared with 2.15 seconds for pypdf.
- A kind of document that may need additional processing: scanned PDFs without a usable text layer may require OCR.

### One thing that went wrong today, and how I fixed it

I initially tried running SQL queries directly in PowerShell, which produced syntax errors because PowerShell does not execute SQL. I corrected this by opening the correct RentRight Neon project and running the queries in the Neon SQL Editor.

### Known discrepancies requiring verification

- The initial parsing run reported 134 pages in PostgreSQL.
- The parser comparison table contains 174 pages when its individual page counts are added.
- D06 reports 266 pages in the collected corpus.
- These figures have different recorded contexts and have not yet been reconciled.
- The cumulative source manifest has 21 unique URLs, whereas the parser comparison covers 10 files.


### D08 · LLM gateway

**Model configuration**

- Primary: `gemini-3.5-flash-lite` (Gemini)
- Fallback: `openai/gpt-oss-20b` (Groq)

**Connectivity test**

Command: `uv run python scripts/llm_ping.py`

- Gemini: OK — `gemini-3.5-flash-lite` responded with Pong in 1465 ms; tokens in = 7, out = 1.
- Groq: OK — `openai/gpt-oss-20b` responded with pong in 753 ms; tokens in = 77, out = 45.

**First question on my laptop**

Question: "Does a tenancy agreement in Tamil Nadu have to be registered?"

- First words after: 3.55 s
- Total response time: 8.46 s
- Tokens in: 183
- Tokens out: 80
- Cost: Rs 0.024216
- Attempts: 1
- Fallback: No

**Repeated refund question**

Question: "Explain in one sentence what a refund is."

First request:
- First words after: 5.11 s
- Total response time: 7.99 s
- Tokens in: 180
- Tokens out: 28
- Cost: Rs 0.011780
- Attempts: 1
- Fallback: No

Second request:
- First words after: 3.35 s
- Total response time: 4.33 s
- Tokens in: 180
- Tokens out: 26
- Cost: Rs 0.011305
- Attempts: 1
- Fallback: No

Additional refund request:
- First words after: 3.48 s
- Total response time: 4.13 s
- Tokens in: 180
- Tokens out: 25
- Cost: Rs 0.011067
- Attempts: 1
- Fallback: No

Day 1 first-word latency target: under 1.5 s. The measured laptop responses have not met this target.

**Live URL**

- First words after: Not measured yet.
- Second, awake request: Not measured yet.

### Tokens: English against Tamil

| Message | Tokens in |
|---|---:|
| "Hi" (mostly system prompt) | 172 |
| English refund sentence | 180 |
| Same sentence in Tamil | 188 |

- English input: 180 tokens.
- Tamil input: 188 tokens.
- Tamil needs approximately 1.04 times the tokens of English (about 4.4% more).

### What the model does without my documents

**Question 6:** "Is my landlord allowed to raise the rent next month?"

The model acknowledged that it could not determine the exact terms of my rental agreement because my specific documents were not connected.

Exact words:

"However, because I do not have your specific documents connected yet, I am not sure about the exact terms in your agreement."

A detail to verify:
- It stated that rent increases generally require proper notice and cannot be arbitrary during an ongoing fixed-term agreement unless agreed upon.
- These legal claims still need verification against the applicable Tamil Nadu law and agreement.

**Off-topic question:**

Question: "Who won the 2011 cricket world cup?"

Response:

"That question is outside this product's subject."

- Result: Successfully declined an unrelated question in one sentence.

### Fallback and circuit breaker

- Primary provider: Gemini.
- Fallback provider: Groq (`openai/gpt-oss-20b`).
- Gemini connectivity: Successful.
- Groq connectivity: Successful.
- Groq ping response time: 753 ms.
- Usage endpoint reports 5 fallbacks and 4 errors.
- Automatic fallback under an intentionally broken Gemini primary: Not yet verified.
- Exact question number at which Gemini error logs stopped appearing: Not yet verified.

### The bill

Command: `Invoke-RestMethod "http://127.0.0.1:8000/v1/usage"`

Usage date: 25 November 2026.

- Total calls: 24
- Errors: 4
- Fallbacks: 5
- Input tokens: 3,939
- Output tokens: 3,675
- Total cost: Rs 0.3903
- Average cost per call: Rs 0.01626
- Day 1 budget target: under Rs 0.50 per answered question.
- Result: Within the cost target, based on the reported 24 calls. Exact average per successfully answered question is not yet established.

### One thing that went wrong today, and how I fixed it

**Problem:** Ruff reported 9 lint errors due to unsorted and unused imports in Python files.

**Fix:**

1. Ran `uv run ruff check . --fix`.
   - All 9 errors fixed.
2. Ran `uv run ruff format .`.
   - 58 files left unchanged.
3. Ran `uv run poe lint`.
   - Ruff check passed.
   - All 58 files were already formatted.

**Additional verification:**

- Created the `llm_calls` database migration using `uv run alembic revision --autogenerate -m "add llm_calls"`.
- Successfully applied it using `uv run alembic upgrade head`.
- Gateway unit tests: 19 passed.
- Full unit tests: 78 passed, 1 warning.
- Local FastAPI server started successfully using `uv run poe dev`.
- Local `/v1/chat` requests returned HTTP 200.
- Gemini success logs appeared with matching request IDs and completed request logs.

### Fallback and circuit breaker

- Primary: Gemini (`gemini-3.5-flash-lite`).
- Fallback: Groq (`openai/gpt-oss-20b`).
- Gemini intentionally broken: Yes.
- Gemini error logged: Yes.
- Automatic Groq fallback: Successful.
- Groq success logged: Yes (`llm call groq ... ok`).
- Request completed successfully: Yes.
- Circuit breaker activation question number: Not yet recorded.
- Gemini recovery after restoring the original key: Not yet verified.
