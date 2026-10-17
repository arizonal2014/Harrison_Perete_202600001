# Portfolio 4 NoSQL and Advanced Data Models

## Document model

The library_events database contains a borrowing_events collection. Each document records one borrowing or return event. Member and copy details are embedded in the event rather than fetched through a SQL join.

| Field | Contents |
| --- | --- |
| _id | Unique event ID, not a loan ID |
| event_type | borrowed or returned |
| occurred_at | Event date and time, stored as a BSON date |
| member | Membership number and name |
| copy | Copy barcode and title |
| details | Due date for a borrowing, or condition for a return |

Example return document:

```javascript
{
  _id: "E002", event_type: "returned",
  occurred_at: ISODate("2026-10-17T12:00:00Z"),
  member: { membership_number: "M001", full_name: "Ada James" },
  copy: { barcode: "LIB002", title: "Things Fall Apart" },
  details: { condition: "good" }
}
```

## Implementation and results

Four fictional documents are inserted through mongosh into the local MongoDB 8.0.34 server. The insert is acknowledged with IDs E001–E004, and countDocuments returns 4. The LIB002 history query returns E001 (borrowed) followed by E002 (returned). Searching details.condition for good returns the complete E002 document, Ada's LIB002 return. These are sample events entered separately, not a complete audit history or an automatic import from PostgreSQL. The SQL database is not changed by this exercise.

## Why MongoDB fits

An event keeps its member, copy and event-specific details together. Borrowing and return details differ without requiring every event to have every field. One document per event lets the history grow without putting all events into one large document. The copied names and titles are snapshots; later SQL changes do not automatically update them. PostgreSQL still handles the lending rules. MongoDB does not enforce the SQL foreign keys or synchronise the two databases here, and this small exercise does not add a schema validator.

## Files

- [MongoDB implementation and queries](Harrison_Perete_202600001_P4_MongoDB.js)
- [Setup and connection check](Harrison_Perete_202600001_P4_Setup.md)
- [Screenshots](screenshots/): MongoDBConnection and EventsAndQueries.

The implementation script stops if borrowing_events already contains documents. It uses printjson to display results when executed as a saved file; it should not be rerun against the populated exercise collection.

## References

- [MongoDB — Data Modeling](https://www.mongodb.com/docs/manual/data-modeling/)
- [MongoDB — insertMany](https://www.mongodb.com/docs/manual/reference/method/db.collection.insertmany/)
- [MongoDB — Query Embedded Documents](https://www.mongodb.com/docs/manual/tutorial/query-embedded-documents/)
