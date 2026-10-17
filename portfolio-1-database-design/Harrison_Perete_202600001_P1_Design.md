# Library Borrowing System Database Design

## My entities

I chose four entities for my library borrowing system:

| Entity | Information stored |
| --- | --- |
| member | A registered borrower |
| book | A catalogue record for a book edition |
| book_copy | One physical copy of a book |
| loan | One borrowing of a physical copy by a member |

I kept book and book_copy separate. For example, the library may have three copies of *Things Fall Apart*. I record the book details once, give each copy its own barcode, and use the loan to record which copy a member borrowed.

### My attributes and keys

I use an automatically generated ID for each record. In the tables below, **unique** means no repeated values. **Required** means the value cannot be `NULL` (missing); checking for blank text is a separate rule.

| Entity | Attributes and proposed PostgreSQL types | Keys and required values |
| --- | --- | --- |
| member | member_id BIGINT; membership_number VARCHAR(30); full_name VARCHAR(120); email VARCHAR(254) | PK: member_id. Unique: membership_number and email separately. All required. |
| book | book_id BIGINT; isbn VARCHAR(17); title VARCHAR(200) | PK: book_id. Unique: isbn. All required. |
| book_copy | copy_id BIGINT; book_id BIGINT; barcode VARCHAR(30) | PK: copy_id. FK: book_id → book.book_id. Unique: barcode. All required. |
| loan | loan_id BIGINT; member_id BIGINT; copy_id BIGINT; borrowed_at TIMESTAMPTZ; due_at TIMESTAMPTZ; returned_at TIMESTAMPTZ | PK: loan_id. FKs: member_id → member.member_id; copy_id → book_copy.copy_id. All required except returned_at. |

`BIGINT` holds whole numbers, such as IDs. `VARCHAR(120)` holds text up to 120 characters. `TIMESTAMPTZ` holds a date and time with time-zone-aware input and display.

I leave returned_at empty until the copy is returned. For example, a new loan has a borrowing date and due date, but no return date yet.

**My assumption:** I record editions with an ISBN, and each ISBN appears once in book.

### My relationships

| Tables linked | Relationship | Meaning |
| --- | --- | --- |
| book → book_copy | One to many | A book record can have zero or more copies; each copy belongs to exactly one book record. |
| member → loan | One to many | A member can have zero or more loans; each loan belongs to exactly one member. |
| book_copy → loan | One to many over time | A copy can have zero or more historical loans; each loan refers to exactly one copy. |

For example, copy 10 can be borrowed in January and again in February. I keep both loan records, but only one can be open at a time. An open loan has no return date. A foreign key links the records; it does not stop the copy being lent twice at the same time.

### How I will apply my rules

- **Rules 1 and 2:** I will prevent repeated membership numbers, emails and copy barcodes using unique constraints.
- **Rules 2 and 3:** I will use required foreign keys so each copy has an existing book, and each loan has an existing member and copy.
- **Rule 4:** I will allow only one open loan per copy. A unique index will check copy_id only where returned_at is missing.
- **Rule 5:** I will allow member_id to repeat in loan so a member can borrow different copies.
- **Rules 6 and 7:** I will check that the due date is after borrowing, and any return date is not before borrowing.
- **Rule 8:** My return operation will update only an open loan and keep its history. I will test this when I implement transactions.

I will find available copies by checking for no open loan. I will find overdue loans by checking for a missing return date and a due date that has passed. I do not need separate availability or overdue fields.
