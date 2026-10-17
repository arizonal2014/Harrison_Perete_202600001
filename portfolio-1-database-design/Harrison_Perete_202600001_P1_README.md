# Portfolio 1 Database Design and Modelling

## Case study

A library borrowing system stores members, book records, physical copies and loan history. Staff issue and return copies, while members borrow them. Staff can check availability and overdue loans.

## Users

Members borrow and return copies. Library staff register members, maintain the catalogue, record loans and returns, and review overdue loans.

## Main operations

Register a member; add a book and its copies; issue a loan; record a return; view available copies, member loan history and overdue loans.

## Business rules

1. Each member has a unique membership number and email address.
2. Each physical copy has a unique barcode and belongs to one book record. A book record can have several copies.
3. Each loan refers to one existing member and one existing copy.
4. A copy can have at most one open loan at a time. An open loan has not been returned.
5. A member can have several open loans, provided each involves a different available copy.
6. The due date must be later than the borrowing date. Staff set the due date when issuing the loan.
7. The return date is empty until return and cannot be earlier than the borrowing date.
8. A return closes an open loan without deleting its history. A loan can be returned only once.

## Scope

One library branch, fictional records and physical book lending. Fines, payments, reservations, author management and user authentication are outside this example.

## Design and implementation files

| File | Contents |
| --- | --- |
| [Design](Harrison_Perete_202600001_P1_Design.md) | Entities, attributes, keys, relationships and business rules. |
| [ER diagram](Harrison_Perete_202600001_P1_ERDiagram.md) | The four tables and their relationships. |
| [Normalisation](Harrison_Perete_202600001_P1_Normalisation.md) | Example tables showing 1NF, 2NF and 3NF. |
| [Schema](Harrison_Perete_202600001_P1_Schema.sql) | PostgreSQL table definitions, keys, date checks and the open-loan index. |
| [Sample records](Harrison_Perete_202600001_P1_SampleData.sql) | Fictional members, a book, physical copies and loans. |
| [Constraint tests](Harrison_Perete_202600001_P1_ConstraintTests.sql) | Six deliberately invalid inserts and the final count query. |
| [Evidence](Harrison_Perete_202600001_P1_Evidence.md) | Links to the screenshots stored in screenshots/. |

## Implementation

The database runs on PostgreSQL 16.15 in an Ubuntu Codespace. Its name is library_portfolio. The schema creates member, book, book_copy and loan. Each has an automatically generated primary key. Foreign keys link copies to books and loans to members and copies.

Membership numbers, emails, ISBNs and copy barcodes are unique. Loan date checks prevent a due date on or before borrowing and a return before borrowing. The loan_one_open_copy index allows at most one open loan per copy while retaining closed loans.

The schema and sample statements were executed in order and committed. The invalid inserts were then run separately, outside a transaction block, to check each rule independently.

## Test results

All six invalid inserts were rejected by the intended rule: a missing member name, repeated email, nonexistent book link, due date before borrowing, return before borrowing and second open loan for LIB001. The screenshots show the corresponding errors.

After the tests, the database contains 2 members, 1 book, 3 copies and 3 loans. One loan is returned and two remain open. The failed inserts added no rows.

These tests cover the listed constraints. The return-once update workflow still needs transaction logic and testing in Portfolio 3; the table definitions alone do not enforce it.

Reference: [PostgreSQL 16 constraints](https://www.postgresql.org/docs/16/ddl-constraints.html).
