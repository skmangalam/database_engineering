## Why Read Committed Allows Race Conditions — PostgreSQL

### 🔄 How Read Committed Works:

* Each **individual query** in a transaction sees a **fresh snapshot** of the latest committed data
* ✅ No dirty reads — you never see uncommitted data
* ❌ But reads are **not repeatable** — the value you read once might change by the time you update, and even though you could re-read to get the latest committed value, you'd have to **continuously re-check** to ensure it hasn’t changed again — which is impractical without locking or versioning

### 🧨 How Race Conditions Happen:

* T1 reads row → sees `balance = 100`
* T2 concurrently updates the same row to `balance = 50` and commits
* T1 now issues an update based on earlier read → sets `balance = 90`
* 💥 T1 has unknowingly **overwritten T2's update**

### ✅ Why it Happens:

* T1 never re-reads the row before updating
* Postgres doesn't track "did someone else modify this between your read and write?"
* So you get **race conditions** and **lost updates**

### ❗Theoretical Fix (But Not Implemented):

> If Postgres had a mechanism to detect that data changed since your read — and auto re-read before write — race conditions could be prevented.

### 🔐 How to Prevent Race Conditions:

| Technique              | What It Does                                     |
|------------------------|--------------------------------------------------|
| `SELECT FOR UPDATE`    | Locks row at read time, blocks concurrent writes |
| Optimistic Locking     | Compare version/timestamp before writing         |
| SERIALIZABLE Isolation | Auto-detects conflicts, forces retry             |

### 💬 Summary:

> **Read Committed gives you clean reads — not safe decisions.** You must add locking or version checks to avoid stale decisions and race conditions.
