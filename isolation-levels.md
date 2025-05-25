## PostgreSQL Transactions, Isolation & MVCC

### ✅ What is a Transaction?

* A transaction is a sequence of SQL operations executed as a **single logical unit of work**.
* Follows **ACID** properties:

    * **Atomicity**: All or nothing
    * **Consistency**: Valid state transitions
    * **Isolation**: Concurrent transactions don’t interfere
    * **Durability**: Once committed, changes survive crashes

---

### 🔒 What is Isolation?

* Defines how **concurrent transactions interact** with shared data.
* Key question: *"Can my transaction see uncommitted changes made by others?"*
* Isolation levels are used to avoid **read anomalies**.

#### 📉 Common Read Anomalies

| Phenomenon              | Description                                                  |
|-------------------------|--------------------------------------------------------------|
| **Dirty Read**          | T1 reads uncommitted data written by T2                      |
| **Non-repeatable Read** | T1 reads same row twice, sees different values               |
| **Phantom Read**        | T1 re-executes a query and sees new rows                     |
| **Lost Update**         | T1 and T2 both update same row; one overwrite goes unnoticed |

---

### 🔄 Isolation Levels in PostgreSQL

| Level            | Dirty Read | Non-repeatable Read | Phantom Read | Lost Update | Snapshot?        | Retry? | Notes                     |
|------------------|------------|---------------------|--------------|-------------|------------------|--------|---------------------------|
| Read Uncommitted | ✅ Yes      | ✅ Yes               | ✅ Yes        | ✅ Yes       | ❌                | ❌ No   | Not truly supported in PG |
| Read Committed   | ❌ No       | ✅ Yes               | ✅ Yes        | ✅ Yes       | Per query        | ❌ No   | Default level             |
| Repeatable Read  | ❌ No       | ❌ No                | ⚠️ Possible  | ✅ Yes       | One snapshot/txn | ❌ No   | Snapshot Isolation        |
| Serializable     | ❌ No       | ❌ No                | ❌ No         | ❌ No        | Snapshot + graph | ✅ Yes  | Highest correctness       |

> PostgreSQL treats `REPEATABLE READ` as **Snapshot Isolation**.

---

### 🧠 What is MVCC?

* **MVCC = Multi-Version Concurrency Control**
* Every row has:

    * `xmin`: transaction that inserted it
    * `xmax`: transaction that deleted/updated it
* Each transaction sees only the versions **valid for its snapshot**

#### 🧪 Example:

* T1 sees balance = 100 (version X)
* T2 updates to 0 (creates version Y)
* T1 still sees X

> MVCC powers non-blocking reads & snapshot isolation — but does **not prevent logical conflicts or lost updates**.

---

### ❌ MVCC ≠ CAS (Compare-And-Swap)

| Feature             | MVCC                       | CAS                     |
|---------------------|----------------------------|-------------------------|
| Visibility Control  | ✅ Yes                      | ❌ No                    |
| Conflict Detection  | ❌ No                       | ✅ Yes (version-based)   |
| Prevent Lost Update | ❌ No (except SERIALIZABLE) | ✅ Yes                   |
| Retry Behavior      | ❌ No                       | ✅ Explicit in app logic |

---

### 🔐 Serializable — Deep Dive

* Simulates serial execution via **predicate locking + dependency graph**
* Automatically **aborts one txn** on write conflicts
* Needs **retry-safe logic** in app layer
* **Deadlocks still possible** due to row lock collisions

#### Retry-safe Pattern:

```java
int retries = 3;
while (retries-- > 0) {
    try {
        beginSerializableTransaction();
        // Read -> business logic -> Write
        commit();
        break;
    } catch (SerializationFailure e) {
        if (retries == 0) throw e;
        Thread.sleep(backoff);
    }
}
```

---

### 💡 Practical Insights

* **MVCC** gives consistent snapshots — but not correctness
* Isolation levels don’t detect *lost updates* — that’s your job (via locks or version checks)
* **Serializable** gives full protection, but needs retry logic
* Read Committed + `SELECT FOR UPDATE` / Optimistic Locking is enough for 90% apps
* Phantom reads occur mostly in **range queries** — SERIALIZABLE handles it
* You **can’t control lock order** in SERIALIZABLE — conflicts are auto-detected

---

### ✅ Summary Decision Table

| Use Case                             | Recommended Isolation       |
|--------------------------------------|-----------------------------|
| Read-only, low consistency need      | Read Committed              |
| Single read → update flow            | Read Committed + FOR UPDATE |
| Multiple reads with logical decision | Repeatable Read             |
| Complex predicate rules              | Serializable                |
| High-concurrency + retry-safe logic  | Serializable                |
| App-managed version conflict         | Optimistic Locking          |
