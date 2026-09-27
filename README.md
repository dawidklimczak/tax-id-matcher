# Tax ID Matcher

A bulk fuzzy-matching tool that resolves a list of customer/company records to their correct Polish tax ID (NIP) against a large local reference database — no live API calls required.

## Problem

An earlier version of this tool (a live lookup against a government API) worked well for a handful of records, but didn't scale to matching tens of thousands of customer records at once: it was slow, rate-limited, and fragile to transient API errors on a long run. The underlying need — "given a messy customer name and address, find the right NIP" — also turned out to be more about fuzzy record linkage than live lookups.

## Solution & Architecture

A shift from a live-API approach to an offline, bulk fuzzy-matching pipeline:

- Loads a large reference database of companies (name, address, NIP) into memory, and pre-builds indexes (by postal code, and by the first characters of the normalized company name) so matching doesn't require comparing every input record against every reference record.
- Runs **two independent matching strategies** per record — name-based (fuzzy string similarity via `rapidfuzz`) and address-based (postal code + city + street + building/apartment, with tiered confidence levels) — and reports whether both agree, since agreement between two independent signals is a strong confidence indicator.
- Normalizes messy real-world input first: Excel's `="0123"` cell-escaping artifacts, inconsistent street-prefix abbreviations (`ul.`, `al.`, `os.`), Polish diacritics, and split building/apartment numbers.
- Is resumable — progress is checkpointed to disk every 50 records, so a large batch can be safely interrupted and continued.
- Reports detailed match-rate and confidence-distribution statistics at the end of a run via a formatted console summary.

## Key Product Decisions

- **Switched from live API lookups to local bulk matching** — a deliberate architecture change once the real workload turned out to be batch record-linkage rather than occasional single lookups; trading real-time freshness for throughput and reliability at scale.
- **Two independent matching signals instead of one blended score** — separating name-match and address-match lets you see *why* a match was made and how confident to be, rather than a single opaque score.
- **Pre-indexing over brute-force comparison** — with reference data in the tens of thousands of rows, indexing by postal code and name prefix keeps matching close to linear instead of quadratic.

## Outcome

Matches large batches of customer records to authoritative company tax IDs without depending on a live government API, with a clear confidence signal per match for manual review of borderline cases.

## Preview

<!-- screenshot placeholder — added separately -->

## Note

Source code is kept in a private repository; happy to share on request.
