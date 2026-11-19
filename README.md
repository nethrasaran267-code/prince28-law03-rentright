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



