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


