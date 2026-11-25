# Dataset: RentRight

Explains tenant and landlord rights under Tamil Nadu's 2017 tenancy law and checks a rental agreement clause by clause.

- **Start target:** TN Act + Rules + Model Tenancy Act
- **Full target:** + portal guides, orders and 50 sample agreements you collect (1,000+ pages)
- **Collected by:** `scripts/fetch.py` as `examate-student-bot` (https://github.com/nethrasaran267-code/prince28-law03-rentright): robots.txt respected, at most 4 requests at once, 0.5 s between requests to one site.

## Where the documents come from

- https://www.tenancy.tn.gov.in/ - Expired
- https://www.indiacode.nic.in/
- https://tnreginet.gov.in/

## Terms I follow

Public law. Cite section numbers.

- Every answer links to the original document.
- No personal data is collected.

<!-- generated:start -->
## Sources (generated 2026-11-25 04:20 UTC by scripts/dataset_report.py)

| Source | Licence / terms | Files | MB | Pages* | How | Fetched | Example URL |
|---|---|---|---|---|---|---|---|
| prs_india | PRS-hosted legal text; verify against official Gazette | 1 | 4.3 | 62 | http | 2026-11-24 | https://prsindia.org/files/bills_acts/acts_states/tamil-nadu/1960/1960TN18.pdf |
| tn_gazette | Government Gazette; verify reuse terms | 6 | 1.1 | 146 | existing_local_file+http | 2026-11-25 | https://www.stationeryprinting.tn.gov.in/extra_ordinary_lists.php?id=MjAxOQ%3D%3D |
| tn_housing_board | Public legal text; verify reuse terms | 2 | 0.8 | 52 | http | 2026-11-24 | https://tnhb.tn.gov.in/assets/documents/Act%20%26%20Rules/79Act%20%282019%29.pdf |
| tn_registration | Government publication; verify reuse terms | 1 | 0.7 | 5 | http | 2026-11-24 | https://tnreginet.gov.in/ |

**Total:** 10 files, 7.0 MB, 266 pages. Not downloaded: 3 blocked by robots.txt, 8 failed.

*Pages = PDF pages, plus one page per 3,000 characters of text or HTML.

## Files (first 10 of 10)

The complete list, with a sha256 fingerprint of every file, is `data/manifest.jsonl`.

| # | Title / file | Source | Pages | Fetched | URL |
|---|---|---|---|---|---|
| 1 | Tamil Nadu Buildings (Lease and Rent Control) Act 1960 | prs_india | 62 | 2026-11-24 | https://prsindia.org/files/bills_acts/acts_states/tamil-nadu/1960/1960TN18.pdf |
| 2 | Tamil Nadu Regulation of Rights and Responsibilities of Landlords and Tenants Act 2017 | tn_gazette | 40 | 2026-11-25 | https://www.stationeryprinting.tn.gov.in/extraordinary/2017/261_Ex_IV_2.pdf |
| 3 | Tamil Nadu Tenancy Rules Amendment 2019 | tn_gazette | 1 | unknown | https://www.stationeryprinting.tn.gov.in/extraordinary/2019/297_Ex_III_1a.pdf |
| 4 | Tamil Nadu Act No. 22 of 2019 - Tenancy Amendment Act | tn_gazette | 3 | unknown | https://www.stationeryprinting.tn.gov.in/extraordinary/2019/311_Ex_IV_2.pdf |
| 5 | Tamil Nadu Tenancy Amendment Ordinance 2019 | tn_gazette | 2 | unknown | https://www.stationeryprinting.tn.gov.in/extraordinary/2019/413_Ex_IV_2.pdf |
| 6 | Tamil Nadu Regulation of Rights and Responsibilities of Landlords and Tenants Rules 2019 | tn_gazette | 12 | 2026-11-25 | https://www.stationeryprinting.tn.gov.in/extraordinary/2019/79_Ex_III_1a.pdf |
| 7 | Tamil Nadu Government Gazette 2019 | tn_gazette | 88 | 2026-11-24 | https://www.stationeryprinting.tn.gov.in/extra_ordinary_lists.php?id=MjAxOQ%3D%3D |
| 8 | Tamil Nadu Regulation of Rights and Responsibilities of Landlords and Tenants Rules 2019 | tn_housing_board | 12 | 2026-11-24 | https://tnhb.tn.gov.in/assets/documents/Act%20%26%20Rules/79Act%20%282019%29.pdf |
| 9 | Tamil Nadu Tenancy Act 2017 | tn_housing_board | 40 | 2026-11-24 | https://tnhb.tn.gov.in/assets/documents/Act%20%26%20Rules/TNRRRLTact2017.pdf |
| 10 | Tamil Nadu Registration Department | tn_registration | 5 | 2026-11-24 | https://tnreginet.gov.in/ |
<!-- generated:end -->

## Known gaps

## Known gaps

- **Start target:** TN Act and Rules need complete source verification; the Model Tenancy Act 2021 is downloaded and text extraction has been validated.
- **Tamil Nadu Gazette:** Only a limited set of Gazette PDFs has been collected; completeness against the relevant tenancy notifications is not yet verified.
- **Portal guides:** Tenancy portal FAQs, registration instructions and user guides are not yet comprehensively collected.
- **Orders:** Relevant Rent Authority and Rent Court orders are not yet collected.
- **Sample agreements:** 50 sample rental agreements are still pending.
- **Coverage:** The full 1,000+ page target has not yet been reached or verified.
