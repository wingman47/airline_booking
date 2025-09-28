# Job Execution
---

### **1. Driver constructs the plan**

* When you call an **action** (like `count()` or `save()`), the **driver**:

  1. Converts your Spark application into a **DAG**.
  2. Breaks the DAG into **stages** (based on shuffle boundaries).
  3. Divides each stage into **tasks** (1 per partition).

---

### **2. ApplicationMaster (AM) requests resources**

* For this Spark application, the **AM**:

  * Requests **containers** (resources) from the **ResourceManager (RM)**.
  * Each container will run a **Spark executor**.

---

### **3. Tasks execution**

* **Driver schedules tasks** from each stage to executors:

  * Tasks in the **same stage** can run in parallel across executors.
  * Stages are executed **sequentially**, but tasks **inside a stage run in parallel**.
  * Data locality is considered when assigning tasks.

---

### **4. Handling resources**

* If more executors are needed → AM asks RM for additional containers.
* Executors are long-running; they can process **many tasks sequentially or in parallel** depending on cores.

---

### **5. Handling failures**

* Task failure → driver reschedules it on another executor.
* Node failure → tasks on that node are rescheduled elsewhere.

---

### **6. Quick Flow Summary**

```
spark-submit → Driver builds DAG → Stages → Tasks
       |
       v
ApplicationMaster requests containers from ResourceManager
       |
       v
NodeManagers launch executors in containers
       |
       v
Executors execute tasks (parallel per stage)
       |
       v
Results sent back to Driver
```

---

✅ Key point:

Jobs (actions) are sequential, stages are sequential, tasks inside a stage are parallel.

Driver builds the plan, AM handles resources, RM allocates containers, NodeManagers run executors, executors run tasks.

Only one NodeManager is chosen to host the AM container.

# Memory

---

## **1. Reserved Memory**

**Definition:**

* Reserved memory is the portion of JVM heap **kept aside by Spark** for internal JVM structures and Spark’s internal operations that must **never be used by tasks**.

**Purpose:**

* Ensures that **critical Spark internals don’t get OOM (OutOfMemoryError)**.
* Used for things like:

  * **Object metadata** (for internal bookkeeping)
  * **Spark’s internal data structures**
  * **JVM overheads like threads, GC, etc.**

**Characteristics:**

* **Fixed size** (default: ~300MB, configurable via `spark.memory.reserved`)
* **Never used by user/executor memory**.
* Protects Spark from crashing due to accidental full memory usage.

**Analogy:**

* Think of it as **“emergency cash”** you don’t touch for regular spending.

---

## **2. Storage Memory Pool**

**Definition:**

* Part of **spark memory pool** (unified memory) used for **caching/persisting RDDs, DataFrames, broadcast variables, shuffle buffers**.

**Purpose:**

* Stores **immutable Spark data** that can be reused across stages without recomputation.
* Helps **avoid recomputation**, improving performance.

**Key Points:**

* Uses **unified memory**, can borrow from execution memory if needed.
* Eviction policy: **Least Recently Used (LRU)** if storage memory exceeds limit.
* If storage memory cannot grow (e.g., too many cached datasets), Spark will **evict old cached blocks**.

**Configuration:**

* `spark.memory.storageFraction` → fraction of unified memory reserved for storage (default 0.5)

**Example Use Case:**

* You cache a DataFrame (`df.cache()`) for multiple transformations. Storage memory holds the cached blocks.

---

## **3. Execution (Executor) Memory Pool**

**Definition:**

* Memory used for **task execution**:

  * Sorting, shuffling, joining, aggregation
  * Intermediate buffers during computation

**Purpose:**

* Ensures **tasks have space to execute without spilling too early**.
* Works dynamically with storage memory if unified memory model is used.

**Characteristics:**

* Part of **unified memory**, can grow into storage memory if storage isn’t using all its share.
* Tasks may spill to **disk** if execution memory is exhausted.
* Includes memory for:

  * Sort-based shuffles (`sort.shuffle`)
  * Aggregation buffers (`aggregateByKey`)
  * Join buffers

**Configuration:**

* Controlled via `spark.memory.fraction` (overall fraction of JVM heap for unified memory) and `spark.memory.storageFraction` (fraction of unified memory for storage).

**Analogy:**

* Think of it as your **workspace on a table**: execution memory is the working area where you perform tasks; storage memory is the **filing cabinet**.

---

## **4. User Memory Pool**

**Definition:**

* Memory available for **user-defined code** in tasks (custom objects, user variables, Spark UDFs).

**Purpose:**

* Allows **developers’ custom logic to safely allocate memory** without crashing Spark internals.
* Includes memory for:

  * User objects in closures
  * Custom caching in UDFs
  * Temporary buffers not part of shuffle/storage

**Key Points:**

* By default, **part of executor JVM heap not used by Spark’s reserved memory**.
* In **unified memory model**, user memory + execution + storage share the same pool indirectly; user memory competes with Spark execution/storage.
* If user code allocates too much, tasks may **OOM**.

**Analogy:**

* Think of it as your **personal backpack**: you can carry whatever objects you want, but you can’t touch the “workspace table” or “filing cabinet” reserved for Spark.

---

**Off-Heap Memory** refers to memory allocated outside the Java Virtual Machine (JVM) heap. Unlike on-heap memory, which is managed by the JVM's garbage collector, off-heap memory allows developers to manage memory allocation directly, thus avoiding GC overhead. This is particularly beneficial for applications that require high performance and low latency, as excessive GC can lead to significant delays in processing.

Off heap memory is used to extend storage memory and executor memory.

---

### **Memory Flow / Summary Table**

| Memory Pool | What it Holds                             | Fixed or Flexible  | Key Points                                                               |
| ----------- | ----------------------------------------- | ------------------ | ------------------------------------------------------------------------ |
| Reserved    | Spark internal structures, JVM overhead   | Fixed (~300MB)     | Never used by tasks; ensures stability                                   |
| Storage     | Cached RDD/DataFrame, broadcast variables | Flexible (unified) | Evicts LRU if full; can borrow from execution memory                     |
| Execution   | Shuffle, sort, aggregation, joins         | Flexible (unified) | Can spill to disk; dynamically borrows from storage memory if available  |
| User        | User objects, closures, UDF buffers       | Flexible (unified) | Can OOM if user code allocates too much; competes with execution/storage |

---

💡 **Important Interview Notes:**

* **Unified memory model** allows **dynamic sharing between execution and storage**.
* **Reserved memory is always protected**.
* **User memory is separate but within JVM heap**.
* **Memory tuning** often revolves around:

  * `spark.executor.memory` (total JVM heap per executor)
  * `spark.memory.fraction` (fraction of JVM heap for unified memory)
  * `spark.memory.storageFraction` (fraction of unified memory for storage)

---

# AQE
---

* **Dynamic Partition Pruning (DPP)**

  * Prunes irrelevant partitions at runtime based on **filter values** from another query.
  * Reduces **I/O and scan cost**.

* **Dynamic Coalescing of Shuffle Partitions**

  * Merges small shuffle partitions **at runtime** based on actual data size.
  * Reduces **task overhead** and improves performance.

* **Dynamic Join Optimization (Join Rewriting)**

  * Converts **large joins into broadcast joins** if one side is small at runtime.
  * Reduces **shuffle and network cost**.

* **Skew Handling**

  * Detects **skewed keys in joins or aggregations** and splits them into smaller tasks.
  * Prevents **long-running straggler tasks**.

* **Query Plan Re-optimization**

  * Updates the **physical plan dynamically** based on **runtime statistics** (like row counts, partition sizes).
  * Helps **choose the best join strategy or partitioning**.

* **Automatic Shuffle Partition Tuning**

  * Adjusts the **number of shuffle partitions dynamically** according to actual data size.
  * Avoids **too many or too few tasks**.

---

# Shuffle
---

## Aggregations

### 🔹 Input RDD

```python
rdd = sc.parallelize([("a", 1), ("a", 5), ("b", 2), ("a", 3)])
```

Let’s assume:

* **Executor 1** has `[("a",1), ("a",5), ("b",2)]`
* **Executor 2** has `[("a",3)]`

---

### 1️⃣ groupByKey

⚙️ **Local behavior**:

* Executor 1: sends raw data → `("a",1), ("a",5), ("b",2)`
* Executor 2: sends raw data → `("a",3)`

⚙️ **Shuffle**:

* All values for key `"a"` are shuffled as-is.
* Reducer receives:

  * `"a"` → `[1, 5, 3]`
  * `"b"` → `[2]`

✅ **Output**:

```python
[("a", [1, 5, 3]), ("b", [2])]
```

📉 **Problem**: 3 separate `"a"` values traveled across the network.

---

### 2️⃣ reduceByKey

⚙️ **Local behavior (map-side combine)**:

* Executor 1: `"a": (1 + 5) = 6`, `"b": 2`
  → local partial results = `[("a",6), ("b",2)]`
* Executor 2: `"a": 3`

⚙️ **Shuffle**:

* Executor 1 sends only `("a",6), ("b",2)`
* Executor 2 sends only `("a",3)`

⚙️ **Reduce phase**:

* `"a"` → `6 + 3 = 9`
* `"b"` → `2`

✅ **Output**:

```python
[("a", 9), ("b", 2)]
```

📈 **Advantage**: Only **two records** for `"a"` crossed the network instead of three.

---

### 🔑 Key Insight

* With **groupByKey**, *every raw value* must shuffle → bigger network cost.
* With **reduceByKey**, *values are partially reduced locally* before shuffle → fewer records sent, faster execution.

---

# Coalescing and Repartition

* **Repartition**

  * Can increase or decrease partitions.
  * Always does a **full shuffle** → balanced data.
  * Use when you need more parallelism or even distribution.

* **Coalesce**

  * Only decreases partitions.
  * **No shuffle** → just merges partitions.
  * Faster but may cause skew.
  * Use before writing fewer output files.

👉 In short:

* **Repartition = shuffle + balance**
* **Coalesce = merge + cheap**


Here’s the updated version of your **bucketing notes** with the mod 4 example added for aggregations:

---

# Bucketing

**Number of buckets** ≈ `Size of dataset in MB / (128–200 MB)`

---

## Bucketing for enhanced filtering

* Use when filtering on a **high-cardinality column**.
* Example: hash the column and assign to buckets (`col % 4`).
* Only the relevant buckets need to be read → **reduces scan time**.

---

## Bucketing for enhanced joins

* If two datasets are **bucketed on the same join key and have the same number of buckets**:

  * Spark can perform **bucketed joins**.
  * Avoids **shuffle**, because rows with the same key are already in the same bucket.
  * Greatly improves **join performance** for large datasets.

---

## Bucketing for enhanced groupBy / aggregations

* When **grouping or aggregating on a bucketed column**:

  * Spark can **process each bucket independently**, reducing shuffle.
* Example: bucket the column using a hash: `col % 4` → 4 buckets.

  * `groupBy` or `reduceByKey` can run **parallel aggregation** per bucket.
  * Only combine results at the final stage → faster aggregation.

---
