# Capacity plan: RentRight

Full corpus target: + portal guides, orders and 50 sample agreements you collect (1,000+ pages)

<!-- generated:start -->
## Measured on my start corpus (2026-11-25 04:20 UTC)

| What | Value |
|---|---|
| Files | 10 |
| Size on disk | 7.0 MB |
| Pages (PDF pages + 1 per 3,000 characters of text/HTML) | 266 |
| Characters of text | 689,235 |
| Estimated tokens (characters / 4) | 172,309 |
| Tokens per page | 649 |
| PDFs with almost no text (scanned? OCR is Day 7) | 0 |
| Files not measured (zip, xlsx, unreadable) | 0 |

## Estimate for the full corpus (target: 1,000 pages)

| What | How | Value |
|---|---|---|
| Scale factor | target / measured | x 3.8 |
| Tokens | measured tokens x scale | 0.65 M |
| Raw files | measured MB x scale | 26 MB |
| Chunks | tokens / 400 x 1.15 overlap | 1,865 |
| Embedding cost, one time | tokens x 1.15 x $0.15 per 1M | $0.11 = Rs 11 |
| Vector storage | chunks x 768 numbers x 4 bytes | 6 MB |
| Text in Postgres | characters x scale x 1.2 | 3 MB |

Assumptions: 4 characters per token, 400-token chunks, 768-dimension embeddings at $0.15 per 1M tokens, Rs 95 per US dollar.
<!-- generated:end -->

## What I conclude

1. Is embedding cost a problem for this product? (1 line, with the number)
2. Do the vectors and text fit in my free Postgres? (1 line: compare with the storage limit on my database dashboard)
3. Which number above am I least sure about, and how will I measure it better? (1 line)
4. What changes if the corpus grows 10 times? (2 lines)


## What I conclude

1. **Embedding cost:** Not a problem; embedding the estimated 0.65 million tokens costs approximately **$0.11 (₹11)** one time.
2. **Postgres storage:** Yes. Estimated vectors (6 MB) and text (3 MB) require approximately 9 MB, well within Neon's 1 GB free storage limit per project.
3. **Least certain number:** Estimated tokens and vector storage, because the current estimate uses 4 characters per token and assumes 768-dimensional embeddings. I will measure actual token counts and embedding sizes using a representative batch.
4. **10× growth:** At 10,000 pages, embedding cost would increase to approximately **$1.10 (₹105)**, with 60 MB of vectors and 30 MB of text.
   Raw files would grow to approximately 260 MB. I would monitor database storage, indexing and retrieval performance as the corpus grows.
