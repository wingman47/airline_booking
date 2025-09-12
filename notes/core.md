# DBMS

### DB Language
DDL (Data Definition Language) → Defines structure of database objects
Examples: CREATE, ALTER, DROP, TRUNCATE

DML (Data Manipulation Language) → Manipulates data inside database objects
Examples: SELECT, INSERT, UPDATE, DELETE

DRL (Retrieve): Select

DCL (Control): Grant, Revoke

TCL (Transaction Control): Commit, Rollback, Savepoint

### ER Model
Entity -> Strong and Weak (cant be uniquely identified)

### Types of attributes:
Simple (cant be divided further, ex - id)
Composit (can be divided, ex - Address)
Single valued (loan amount)
Multi-valued (phone numer)
Derived (age)
null value

### Relationship constraints
one to One (person to aadhar)
one to many (person to phone no.)
many to one (person to area pin code)
many to many (Customer buys product, M no. of customers buy N no. of Products)

### Keys:
-	Super Key: Any unique identifier (can have extra attributes).
-	Candidate Key: Minimal super key (no extra attributes).
-	Primary Key: Candidate key chosen as main identifier.
-	Composite Key: PK formed using at least 2 attributes.

### Key contraints:
not null
unique
default
primary key
foreign key
check, ex: email TEXT CHECK (email LIKE '%@%'),       

### Normalisation
https://chatgpt.com/share/68c18ebd-5d88-8008-a6f7-d77dca0c0a5f
https://arpit-anand.notion.site/SQL-1e3e833878a18080adb3d5a44402528b

### ACID: atomicity, consistency, isolation, durability
### Atomicity and durability implementation

#### Using shadow copy
db pointer points to current copy in the db
shadow copy is created before execution of a transaction T
if T is successfull on shadow copy 
- DB system updates the db-pointer to point to the new copy of DB.
- new copy is now the original copy
- old copy is deleted
- transaction is committed
- inefficient since whole db copy for each T

#### Log based
Ensuring atomicity by recording all the DB modifications in the log but deferring the execution of all the write operations
until the final action of the T has been executed.
2. Log information is used to execute deferred writes when T is completed.
3. If system crashed before the T completes, or if T is aborted, the information in the logs are ignored.
4. If T completes, the records associated to it in the log file are used in executing the deferred writes.
5. If failure occur while this updating is taking place, we preform redo.
This version is close, but still has a couple of terminology slips that can cause confusion. Let me fix it into a clean, exam-ready minimal summary:

# Indexing

There are 2 types of indexing (based on how the data file is stored):

### 1. Primary Indexing

* File is sorted on the primary key (unique).
* Sparse index is used → one entry per block (since key is unique and ordered).

### 2. Clustering Indexing

* File is sorted on a non-key attribute (duplicates allowed).
* Index has one entry per distinct attribute value.
* Records with the same value are physically clustered together.
* Example:

  ```
  (CS → ptr1)
  (EC → ptr2)
  (IT → ptr3)
  ```

  Searching `Dept=CS` → binary search in index → jump to first CS record, then scan sequentially.

### Dense vs Sparse Index

* Dense Index → one entry per record (direct pointer).
* Sparse Index → one entry per block/range (need to search inside block).

### Multi-Level Indexing (Not treated as separate)

* Index on the index (hierarchical).

### Secondary Indexing (Non-Clustering Index)

* File is unsorted, so primary/clustering index not possible.
* Can be built on key or non-key attributes.
* Always dense (one entry per record).

### DB Clustering

* Data redundancy
* Load balancing
* High availability

### CAP theorem
* Consistency: In a consistent system, all nodes see the same data simultaneously. If we perform a read operation on a consistent system, it
should return the value of the most recent write operation. The read should cause all nodes to return the same data. All users see the same data
at the same time, regardless of the node they connect to.
* Availability: 
  response guarantee (user always gets an answer). 
  Even if some nodes are down, the system gives some answer.
  Doesn’t guarantee it’s the latest data.
* Partition tolerance: survival guarantee (system doesn’t crash during network splits).

Partition = Network Split

Some nodes can’t talk to others (messages are lost, delayed, or dropped).
The system is divided into two or more “islands” of nodes that keep running but can’t fully sync.
This is not about nodes failing, but about the links between them failing.
Example:
Imagine a distributed database with nodes in Mumbai and Delhi.
If the network link between the two cities breaks, Mumbai users only see Mumbai nodes, Delhi users only see Delhi nodes.
Both groups can still use the system, but they might see different versions of the data until the partition heals.
─────────────────────────────
👉 So, in CAP, Partition Tolerance (P) means the system can continue operating correctly even if such splits happen.

### 🔹 Why we can’t have all 3

* Partition tolerance (P) is non-negotiable in distributed systems — networks can fail, nodes can go down, messages can be delayed.
* So we must always tolerate partitions → leaving only CA or AP or CP choices.
* The real conflict comes when a partition happens:
  → Do you return possibly stale data (choosing Availability)
  → Or do you block requests until consistency is restored (choosing Consistency)?

---

### 1. CA (Consistency + Availability, but no Partition Tolerance)

* You can’t survive partitions here.
* Why ❌? Because the moment a partition happens, if you want C + A together:

  * Consistency demands *all nodes see the same data*.
  * Availability demands *every request gets a response*.
  * But if the cluster splits, some nodes can’t talk → you cannot guarantee both without violating P.
* Example: A single-node relational DB running on one machine (no partitions possible). Works fine for CA, but not distributed.

---

### 2. AP (Availability + Partition Tolerance, but no Strong Consistency)

* You choose to keep the system running during partitions and always respond.
* Why ❌ Consistency? Because:

  * If partitions occur, some nodes may have stale or divergent data.
  * You serve “whatever version you have” to stay available.
  * System becomes eventually consistent, not strongly consistent.
* Example: DynamoDB, Cassandra, CouchDB → prioritize AP.

---

### 3. CP (Consistency + Partition Tolerance, but no Availability)

* You keep data consistent across nodes even when partitions happen.
* Why ❌ Availability? Because:

  * To preserve consistency, some nodes must refuse requests until they can sync with the others.
  * That means some requests fail (unavailable) during the partition.
* Example: HBase, MongoDB (with majority writes), ZooKeeper → prioritize CP.

## Summary

db language:
ddl
dml
drl
tcl
dcl

attributes:
single
complex
null
derived
multi-valued

keys:
super
candidate (minimal attributes for pk)
primary
foreign

key constraints:
check
not null
foreign
primary

default
unique

1nf:
not multi-valued

2nf:
1nf + no partial dependency
ex: we have - order_id, item_id, customer_id
if pk is item_id + customer_id and order_id is only dependent on 
customer_id then not in 2nf

3nf:
2nf + no transitive dependency
ex: 
order_id (pk), customer_id, customer_name, customer_email

# OOP

### Singleton class
```java
class Singleton {
    private static Singleton instance;
    private Singleton() {
        
    }
    private static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton()
        }
        return instance;
    }
}
```

### Static block
The code inside the static block is executed when the class is loaded for the first time. Therefore it is only executed once.
```java
class Vehicle {
    static {
        System.out.println("Vehicle object initialized");
    }
}
```

- Static methods can only use static variables and instances (use ```static class ClassName {}```)
- Static variables are shared among all instances
- super() calls parent constructor
- this() calls the previous constructor of the same class and if that class also uses this() then previous class is also called

## Inheritance

- When we don’t use any keyword explicitly, Java will set a default access to a given class, method, or property. The default access modifier is also called package-private, which means that all members are visible within the same package, but aren’t accessible from other packages. 
- Subclasses inherit fields and methods from the superclass and can access or override them. Child classes cannot access private properties of parent class.
- All the parent constructors are called when an object is initialized.
- super() can only be used in constructor as the first statement.
```java
// class BoxWeight extends Box {}
BoxWeight box = new Box(); // Error: cannot reference a child to a parent
// vice versa is allowed
```
- If we add a constructor in a class then we must provide a default (parameter-less) constructor to the class otherwise you will get errors.

### Types of inheritance
- Simple
- Multilevel (as the name suggests)
- Hierarchical (multiple classes with same parent)
- Multiple (using interfaces)
- Cyclic (derived from itself through a chain)

## Polymorphism
- runtime: method/constructor/operator overloading
- compile time: method overriding (we can prevent it by using final keyword)

## Encapsulation
- involves restricting access to some of an object's components and preventing the direct modification of internal details

## Abstraction
- Abstraction is the process of hiding the implementation details and showing only the essential features of an object. It allows you to focus on what an object does rather than how it achieves its functionality. 
- abstraction deals with the design level issues like selecting which methods and variables to provide to the user while encapsulation deals with the actual implementation.
```java
abstract class Animal {
    String type = "Unknown";   // normal variable

    abstract void sound();     // abstract method
}

class Dog extends Animal {
    // must implement abstract method
    void sound() {
        System.out.println("Bark");
    }
}

public class Test {
    public static void main(String[] args) {
        Dog d = new Dog();
        d.sound();                   // Bark
        System.out.println(d.type);  // Unknown
    }
}
```

## Interface
- An interface defines only the what (method signatures), not the how.
- Abstract classes can mix abstract methods + concrete methods, but they may also carry state (fields).
- A class can implement multiple interfaces, which gives it the ability to inherit behavior contracts from multiple sources.
- Can only have final public static variables so variables always needs to be initialized.
- We can now provide default methods in interface which is like concrete methods in abstract classes.
- It's mandatory to implement all the methods.
```java
interface A {
    void methodA();
}

interface B {
    void methodB();
}

class C implements A, B {
    public void methodA() {
        System.out.println("Method from A");
    }
    public void methodB() {
        System.out.println("Method from B");
    }
}

public class Test {
    public static void main(String[] args) {
        C obj = new C();
        obj.methodA();
        obj.methodB();
    }
}
```

## Packages
- java.lang: This package is automatically imported in every Java program. It contains fundamental classes such as Object, String, Math, and basic data types (int, String, etc.).
- We can create a package by moving the class file inside a folder <package_name> and on the first line write ```package <package_name>``` 
  and we can import it using <package_name>.<class_name>
- We can also create nested folders for packages.

## 1. Encapsulation

* ATM Machine → You just press buttons; the internal cash-dispensing mechanism is hidden.
* Car Dashboard → You see speed, fuel, etc. the engine’s internal working is hidden from you.

## 2. Inheritance

* Vehicle Hierarchy → Car, Bike, Truck all inherit common features like wheels, engine, brakes.
* Electronics → Smartphone, Laptop, Tablet all inherit common properties like battery, screen, processor.

## 3. Polymorphism

* Remote Control → Same button (“power”) works differently for TV, AC, or Music System.
* Human Communication → The word “run” can mean running a race, running a program, or running a company (different meanings in different contexts).

## 4. Abstraction

* Coffee Machine → You press “Latte” and it makes it; you don’t know the internal grinding/boiling process.
* Online Payment → You just click “Pay”. the system hides complex bank verifications and API calls.

# SQL

```sql
INSERT INTO employees (id, name, salary, department_id)
VALUES (1, 'Alice', 60000, 1);

-- Update
UPDATE employees SET salary = 70000 WHERE id = 1;

-- Delete
DELETE FROM employees WHERE id = 1;

-- Constraints
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    salary DECIMAL(10,2) CHECK (salary > 0),
    department_id INT UNIQUE
);
```