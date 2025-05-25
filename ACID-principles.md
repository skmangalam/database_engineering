### what is a transaction?

- A transaction is a collection of queries that are executed as a single logical unit of work
- It follows the ACID properties:
  - ATOMICITY - all operations succeed or none at all
  - ISOLATION - concurrent transactions don't interfere with each other
  - CONSISTENCY - DB moves from one valid state to another
  - DURABILITY - once committed, changes are permanent even after crash

### what is isolation?

- can my inflight transaction see changes being made by other inflight transactions?
- as a result of these, we get a bunch of read phenomenas - these are undesirable
- to solve these problems - isolation levels were introduced


### Read phenomena

- Dirty reads : a transaction reads something which some other transaction just wrote but did not really commit yet
- Non-repeatable reads : a transaction reads something and then re-read it again, but the value has changed
- Phantom reads : a transaction did not read it initially because it did not exist, but then when reading again you encountered it
- Lost updates : a transaction wrote something but some other transaction modified this value, so what you wrote essentially got lost

❓ What is Isolation?
Isolation defines how the database handles concurrent transactions — especially whether they can see each other’s intermediate states.

In simple terms:
👉 "Can my in-flight transaction see changes made by other in-flight transactions?"

Low isolation = more concurrency but less safety
High isolation = less concurrency but more safety

To deal with concurrency issues, databases define Isolation Levels.

### Read Phenomena (Problems Isolation Solves)

| Phenomenon          | What Happens                                                                   |
|---------------------|--------------------------------------------------------------------------------|
| Dirty Read          | T1 reads data written by T2 which is not committed yet                         |
| Non-repeatable Read | T1 reads the same row twice, gets different values (because T2 updated it)     |
| Phantom Read        | T1 runs `SELECT` twice — in the second run, new rows “appear” (inserted by T2) |
| Lost Update         | T1 writes a value, but T2 also writes over it — T1's write gets lost silently  |


### 📊 SQL Isolation Levels and What They Prevent

| Isolation Level  | Prevents Dirty Read | Prevents Non-repeatable Read | Prevents Phantom Read |
|------------------|---------------------|------------------------------|-----------------------|
| Read Uncommitted | ❌ No                | ❌ No                         | ❌ No                  |
| Read Committed   | ✅ Yes               | ❌ No                         | ❌ No                  |
| Repeatable Read  | ✅ Yes               | ✅ Yes                        | ❌ No                  |
| Serializable     | ✅ Yes               | ✅ Yes                        | ✅ Yes                 |


**Higher isolation gives stronger consistency guarantees, but can hurt performance due to locking and reduced concurrency**

