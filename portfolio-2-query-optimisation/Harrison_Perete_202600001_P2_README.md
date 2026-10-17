# Portfolio 2 Query Processing and Optimisation

## Queries and results

The open-loan query returns Ben Cole with LIB001, due October 22, 2026, and Ada James with LIB002, due October 8, 2026. The availability query returns LIB003, a copy of Things Fall Apart. The grouped query counts two loans for Ada and one for Ben, including returned loans.

## Index comparison

The separate p2_benchmark.loan_history table contains 100,000 generated, closed loans using the existing member M001 and copy LIB001. The Portfolio 1 records remain unchanged. This test table is created with CREATE TABLE AS and does not copy the operational table's constraints.

The same query counts loans borrowed from January 1, 2005 (inclusive) to January 2, 2005 (exclusive), using explicit UTC timestamps. It returns 24 before and after adding loan_history_borrowed_at_idx on borrowed_at. ANALYZE refreshes the test table statistics before measurement. Each phase has one warm-up and three measured EXPLAIN (ANALYZE, BUFFERS) runs.

| Measurement | Before index | After index |
| --- | --- | --- |
| Scan type | Seq Scan | Index Only Scan |
| Three execution times (ms) | 7.941, 5.534, 14.668 | 0.039, 0.040, 0.030 |
| Median time (ms) | 7.941 | 0.039 |
| Shared buffer hits, third run | 960 | 3 |

## Performance analysis

Before indexing, PostgreSQL scans 100,000 rows and removes 99,976 through the date filter. After indexing, the index-only scan finds the 24 matching entries, with zero heap fetches in these measured runs. The median execution time falls from 7.941 ms to 0.039 ms and buffer hits fall from 960 to 3. Both phases use cached data; the variable before-index times also show that this shared Codespace is not a controlled production benchmark. The result applies to this narrow date range, not every query. An index uses extra storage and adds maintenance work when records change.

## Files

- [SQL queries](Harrison_Perete_202600001_P2_Queries.sql)
- [Test data, execution plans and index SQL](Harrison_Perete_202600001_P2_Optimisation.sql)
- [Screenshots](screenshots/): QueryResults, BeforeIndex and AfterIndex.

## Reference

[PostgreSQL 16 — Using EXPLAIN](https://www.postgresql.org/docs/16/using-explain.html).
