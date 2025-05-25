## Why Repeatable Read Allows Lost Updates — PostgreSQL

### 🔄 How Repeatable Read Works:

* When a transaction begins under `REPEATABLE READ`, PostgreSQL assigns it a **snapshot** of the database at that moment.
* All reads within that transaction refer to **that same snapshot**, regardless of concurrent commits by other transactions.
* This ensures **read stability** — every SELECT sees the same consistent state.

### 🧨 How Lost Updates Can Still Happen:

* T1 begins and reads balance = 100 from its snapshot.
* While T1 is still running, T2 commits an update: balance = 80.
* T1, unaware of T2's change, updates balance to 90 (based on its old snapshot).
* PostgreSQL allows this because T1's update is valid **from its snapshot's perspective**.
* The result: **T1 silently overwrites T2’s change** — a lost update.

### ✅ Why It Happens:

* PostgreSQL does **not compare the current value with what T1 read earlier**.
* It only checks if the row being updated was visible in T1’s snapshot.
* No version match check = no conflict detection at write time.

### 🔐 How to Prevent Lost Updates:

| Technique              | How It Helps                                |
|------------------------|---------------------------------------------|
| `SELECT FOR UPDATE`    | Locks the row; blocks concurrent writers    |
| Optimistic Locking     | Use version columns and conditional updates |
| SERIALIZABLE Isolation | Detects and aborts conflicting transactions |

### 💬 Summary:

> **Repeatable Read ensures consistent views for reading, but does not protect against other committed transactions modifying the same data in the background.** If the application does not use locking or version checking, lost updates are still possible.
