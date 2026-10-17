# Portfolio 3 Transactions and Concurrency

## Commit and rollback

The return for LIB002 is committed with a return date of October 17, 2026. A repeated return affects zero rows because the UPDATE only changes a loan whose returned_at is NULL. In the next transaction, a valid LIB003 loan is added before an invalid due date raises a loan_due_after_borrowing error. ROLLBACK removes that new loan; the earlier committed return remains. The checks show a saved return of 2026-10-17 and zero LIB003 loans at that point.

## Isolation and concurrency control

Two PostgreSQL sessions use READ COMMITTED. A inserts a LIB003 loan for Ben without committing. B sees zero open loans, then tries to insert one for Ada. B waits, with wait_event_type shown as Lock. After A commits, B receives a loan_one_open_copy duplicate-key error. B rolls back, and the final query shows exactly one open LIB003 loan, belonging to Ben. A's COMMIT and B's ROLLBACK finish both transactions.

## Conclusion

COMMIT saves changes and ROLLBACK undoes a transaction's unsaved changes. READ COMMITTED hides another session's uncommitted records. The unique partial index prevents two open loans even when an earlier availability check finds none. The return-once check belongs to the demonstrated UPDATE workflow; it does not prevent every possible direct database edit. These tests change the live library records, while the earlier portfolio screenshots record earlier states. The separate Portfolio 2 speed-test table is unchanged.

## Files

- [Commit and rollback SQL](Harrison_Perete_202600001_P3_Transactions.sql)
- [Session A SQL](Harrison_Perete_202600001_P3_SessionA.sql)
- [Session B SQL](Harrison_Perete_202600001_P3_SessionB.sql)
- [Screenshots](screenshots/): CommitRollback, SessionsWaiting and ConcurrencyResult.

Run the two-session examples in the codelab's A → B → A → B order, not as two complete scripts executed sequentially.

## References

- [PostgreSQL 16 — Transactions](https://www.postgresql.org/docs/16/tutorial-transactions.html)
- [PostgreSQL 16 — Transaction Isolation](https://www.postgresql.org/docs/16/transaction-iso.html)
- [PostgreSQL 16 — Unique Indexes](https://www.postgresql.org/docs/16/indexes-unique.html)
