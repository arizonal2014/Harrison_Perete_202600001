# Library Borrowing System Database Design

## Entities

The design uses four entities:

| Entity | Information stored |
| --- | --- |
| member | A registered borrower |
| book | A catalogue record for a book edition |
| book_copy | One physical copy of a book |
| loan | One borrowing of a physical copy by a member |

Book and book_copy are separate because several physical copies can share the same title and ISBN. A loan identifies the physical copy that was borrowed.

## Choose attributes and keys

A **primary key (PK)** identifies a row. A **foreign key (FK)** links it to a row in another table. **Unique** means a value cannot be repeated in that column. **Required** means the value cannot be empty (`NULL`).

| Entity | Attributes and proposed PostgreSQL types | Keys and required values |
| --- | --- | --- |
| member | member_id BIGINT; membership_number VARCHAR(30); full_name VARCHAR(120); email VARCHAR(254) | PK: member_id. Unique: membership_number and email separately. All required. |
| book | book_id BIGINT; isbn VARCHAR(17); title VARCHAR(200) | PK: book_id. Unique: isbn. All required. |
| book_copy | copy_id BIGINT; book_id BIGINT; barcode VARCHAR(30) | PK: copy_id. FK: book_id → book.book_id. Unique: barcode. All required. |
| loan | loan_id BIGINT; member_id BIGINT; copy_id BIGINT; borrowed_at TIMESTAMPTZ; due_at TIMESTAMPTZ; returned_at TIMESTAMPTZ | PK: loan_id. FKs: member_id → member.member_id; copy_id → book_copy.copy_id. All required except returned_at. |

The primary-key IDs will be generated automatically. `TIMESTAMPTZ` stores a date and time representing an instant, with time-zone-aware input and display.

**Additional design assumption:** each catalogue record represents one ISBN-bearing edition, and its ISBN is unique. This design covers ISBN-bearing editions.

## Describe the relationships

| Parent → child | Relationship | Meaning |
| --- | --- | --- |
| book → book_copy | One to many | A book record can have zero or more copies; each copy belongs to exactly one book record. |
| member → loan | One to many | A member can have zero or more loans; each loan belongs to exactly one member. |
| book_copy → loan | One to many over time | A copy can have zero or more historical loans; each loan refers to exactly one copy. |

Historical loans can be numerous, but a copy can have only one open loan at a time. This needs an additional constraint; the foreign key alone does not prevent two simultaneous loans.

## Connect the rules to the design

- Unique membership numbers, emails and barcodes address Rules 1 and 2.
- Required foreign keys address the links in Rules 2 and 3.
- A unique index on copy_id for loans with no return date will address Rule 4.
- No unique constraint on member_id in loan allows Rule 5.
- Date checks will enforce due_at > borrowed_at and returned_at ≥ borrowed_at when returned_at is present, addressing Rules 6 and 7.
- The return operation will update only an open loan and retain its row, addressing Rule 8. This behaviour needs transaction logic and testing, not just a column definition.

Availability comes from the absence of an open loan. An overdue loan has no return date and its due time has passed. Separate stored availability and overdue flags are unnecessary for this design.
