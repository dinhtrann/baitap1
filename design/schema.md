# Database Schema - Library Management System

## Entity-Relationship Diagram

```mermaid
erDiagram
    auth_users ||--o| profiles : "has"
    profiles ||--o{ borrows : "creates"
    profiles ||--o{ return_requests : "creates"
    profiles ||--o{ penalties : "receives"
    profiles ||--o{ borrows_approved : "approves"
    categories ||--o{ books : "categorizes"
    books ||--o{ borrows : "borrowed_in"
    books ||--o{ return_requests : "returned_in"
    borrows ||--o| return_requests : "has"
    borrows ||--o{ penalties : "generates"
    return_requests ||--o{ penalties : "generates"
    penalty_levels ||--o{ penalties : "defines"

    auth_users {
        uuid id PK
        string email
        timestamp created_at
    }

    profiles {
        uuid id PK
        uuid user_id FK "UNIQUE, references auth.users"
        varchar full_name
        varchar phone "nullable"
        varchar address "nullable"
        enum role "reader, librarian, admin"
        enum status "active, inactive"
        integer total_borrows "default 0"
        numeric total_penalties "default 0"
        timestamp created_at
        timestamp updated_at
    }

    categories {
        uuid id PK
        varchar name "UNIQUE, max 50"
        timestamp created_at
        timestamp updated_at
    }

    books {
        uuid id PK
        varchar title "max 100"
        varchar author "max 100"
        varchar isbn "nullable, UNIQUE"
        integer publication_year
        uuid category_id FK
        text description "max 500"
        integer available_quantity "default 0"
        integer borrowed_quantity "default 0"
        enum status "available, damaged, lost"
        timestamp created_at
        timestamp updated_at
    }

    borrows {
        uuid id PK
        uuid user_id FK "references profiles"
        uuid book_id FK "references books"
        uuid librarian_id FK "nullable, references profiles"
        integer borrow_duration_days "default 14, max 30"
        timestamp borrow_date
        timestamp due_date
        timestamp return_date "nullable"
        enum status "pending, approved, rejected, borrowed, returned, overdue"
        text rejection_reason "nullable, max 500"
        integer extension_count "default 0, max 1"
        timestamp extended_due_date "nullable"
        timestamp created_at
        timestamp updated_at
    }

    return_requests {
        uuid id PK
        uuid borrow_id FK "references borrows, UNIQUE when status=pending"
        uuid book_id FK "references books"
        uuid user_id FK "references profiles"
        timestamp request_date
        enum status "pending, confirmed"
        enum return_condition "normal, damaged, lost"
        text notes "nullable, max 500"
        timestamp created_at
        timestamp updated_at
    }

    penalty_levels {
        uuid id PK
        varchar name "max 25"
        numeric amount "> 0"
        date effective_date
        timestamp created_at
        timestamp updated_at
    }

    penalties {
        uuid id PK
        uuid user_id FK "references profiles"
        uuid borrow_id FK "nullable, references borrows"
        uuid return_request_id FK "nullable, references return_requests"
        uuid penalty_level_id FK "references penalty_levels"
        enum type "late_return, damaged, lost"
        numeric amount
        enum status "unpaid, pending, paid, rejected"
        text notes "nullable, max 500"
        text rejection_reason "nullable, max 500"
        timestamp payment_date "nullable"
        timestamp created_at
        timestamp updated_at
    }
```

## Indexes

### Primary Indexes (Automatic)
- All tables have UUID primary keys

### Foreign Key Indexes
- `profiles.user_id` (unique index)
- `profiles.id` (for borrows.user_id, return_requests.user_id, penalties.user_id)
- `books.category_id`
- `books.id` (for borrows.book_id, return_requests.book_id)
- `borrows.id` (for return_requests.borrow_id, penalties.borrow_id)
- `borrows.user_id` (for user's borrow history queries)
- `borrows.book_id` (for book's borrow history queries)
- `borrows.status` (for filtering by status)
- `return_requests.borrow_id` (unique partial index for pending status)
- `return_requests.status` (for filtering pending returns)
- `penalties.user_id` (for user's penalty queries)
- `penalties.status` (for filtering unpaid/pending penalties)
- `penalty_levels.id` (for penalties.penalty_level_id)

### Composite Indexes
- `borrows(user_id, status)` - for user's active borrows
- `borrows(book_id, status)` - for book availability checks
- `penalties(user_id, status)` - for user's unpaid penalties
- `books(category_id, status)` - for category filtering
- `books(title, author)` - for search queries

### Unique Constraints
- `profiles.user_id` - one profile per auth user
- `categories.name` - unique category names
- `books.isbn` - unique ISBN (when provided)
- `return_requests(borrow_id)` - partial unique index where status = 'pending'

## Relationships Summary

1. **auth_users → profiles**: One-to-One (via user_id)
2. **profiles → borrows**: One-to-Many (user creates borrows)
3. **profiles → return_requests**: One-to-Many (user creates return requests)
4. **profiles → penalties**: One-to-Many (user receives penalties)
5. **profiles → borrows (as librarian)**: One-to-Many (librarian approves/rejects)
6. **categories → books**: One-to-Many (category has many books)
7. **books → borrows**: One-to-Many (book can be borrowed many times)
8. **books → return_requests**: One-to-Many (book can be returned many times)
9. **borrows → return_requests**: One-to-One (one return request per borrow when pending)
10. **borrows → penalties**: One-to-Many (borrow can generate multiple penalties)
11. **return_requests → penalties**: One-to-Many (return can generate penalties)
12. **penalty_levels → penalties**: One-to-Many (level defines many penalties)

## Business Rules Enforced

1. **User Registration**: New users start with status 'inactive' (pending approval)
2. **Borrow Limits**: Maximum 5 active borrows per user (enforced in application)
3. **Borrow Duration**: Default 14 days, maximum 30 days
4. **Extension**: Maximum 1 extension per borrow (+7 days)
5. **Return Request**: Only one pending return request per borrow
6. **Penalty Types**: late_return, damaged, lost
7. **Penalty Status Flow**: unpaid → pending → paid/rejected

