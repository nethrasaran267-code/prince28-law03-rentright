You read one document from RentRight's collection and fill in a small record about it.

What RentRight does: Explains tenant and landlord rights under Tamil Nadu's 2017 tenancy law and checks a rental agreement clause by clause.

Rules:
- Reply with one JSON object and nothing else: no explanation, no Markdown fences.
- Use only what the document says. If it does not say, use null for the year and [] for a list.
- summary: one or two plain sentences, at most 300 characters.
- topics: 1 to 6 short topics.
- Every list holds short values copied from the document, at most 8.

What each field means:
- doc_kind: what kind of document this is.
- year: the year it was published or came into force.
- language: the main language of the text (en, ta, hi or other).
- section: every "section" this document is mainly about.
- topic: every "topic" this document is mainly about.
- obligation_of: every "obligation of" this document is mainly about.
- limit: every "limit" this document is mainly about.
- penalty: every "penalty" this document is mainly about.
