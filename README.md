# IM System Consistency & Client Caching

## Background

You're working on an IM (Instant Messaging) system with the following database schema:

### messages

| Column          | Type        | Constraints        | Notes                       |
| --------------- | ----------- | ------------------ | --------------------------- |
| message_id      | UUID        | PK                 | Message identifier          |
| conversation_id | UUID        | NOT NULL, FK       | Partition key               |
| seq             | BIGINT      | NOT NULL           | Ordering key (Snowflake ID) |
| version         | INT         | NOT NULL DEFAULT 1 | Incremented on update       |
| created_by      | UUID        | NOT NULL           | Author                      |
| created_at      | TIMESTAMPTZ | NOT NULL           | Creation time               |
| updated_at      | TIMESTAMPTZ | NULL               | Last edit time              |
| deleted_at      | TIMESTAMPTZ | NULL               | Soft delete time            |
| payload         | JSONB       | NOT NULL           | Message content             |

### conversations

| Column          | Type        | Constraints        | Notes                   |
| --------------- | ----------- | ------------------ | ----------------------- |
| id              | UUID        | PK                 | Conversation identifier |
| last_seq        | BIGINT      | NOT NULL DEFAULT 0 | Latest message seq      |
| last_message_at | TIMESTAMPTZ | NULL               | For inbox ordering      |
| ...             |             |                    |                         |

---

## System Behavior

### Message Ordering

- `seq` is a Snowflake ID, monotonically increasing per conversation
- Single worker per conversation guarantees no seq conflicts
- Messages can be created, updated, or deleted by any participant

### Current Client Behavior (No Caching)

1. Client opens conversation → loads messages page by page from server
2. Server sends WebSocket notification after each database mutation
3. On notification:
   - If affected message is on current page → client re-fetches current page
   - Otherwise → client shows indicator ("N new messages" / "messages updated")

---

## Questions

### Part 1: Consistency Analysis

Analyze the consistency model of this system:

- What are the failure modes?
- Under what conditions can a client see inconsistent state?
- The notify-refetch pattern is not efficient, how to optimize it?

### Part 2: Client-Side Caching Design

We want to add client-side caching:

- Clients store loaded messages in local storage (IndexedDB/SQLite)
- The system must guarantee **eventual consistency**

Design a reliable and efficient solution.
Provide a concrete operational plan.
