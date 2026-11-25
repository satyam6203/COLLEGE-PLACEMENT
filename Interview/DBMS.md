🗄️ DBMS Interview Questions (1–100)

> Format: **Answer** → **Example** → **Interview Follow-up** → **Short Interview Answer**

---

## 1. What is DBMS?

**Answer:**
DBMS stands for Database Management System — software that stores, retrieves, updates and manages data on behalf of users. It provides CRUD operations, security, concurrency control, backup and recovery, so I don't have to manage files manually. MySQL, PostgreSQL, Oracle and MongoDB are all DBMSs.

**Example:**
When a Spring Boot app calls `JdbcTemplate` or Hibernate, they talk to the DBMS, which actually reads/writes the data files.

**Interview Follow-up:**
- Which DBMS have you used in production?
- Is MongoDB a DBMS? (Yes, it's a NoSQL DBMS.)

**Short Interview Answer:**
A DBMS is software that manages data efficiently — it handles storage, security, concurrency and backup so applications can simply run queries.

---

## 2. How is DBMS different from a file system?

**Answer:**
A file system stores data as separate files with no relationships or constraints between them, while a DBMS stores data in related tables with defined structure and rules. In a file system we manually write code for every lookup and update; in a DBMS the SQL does it for us. The biggest difference is that a DBMS removes redundancy and keeps data consistent automatically.

| Feature | File System | DBMS |
|---|---|---|
| Structure | Loose files | Tables with schema |
| Redundancy | High, duplicated | Low, controlled by normalization |
| Consistency | Manual | Enforced by constraints |
| Security | OS-level (weak) | User roles, GRANT/REVOKE |
| Queries | Custom code per app | SQL |
| Backup/Recovery | Manual | Built-in (WAL, checkpoints) |
| Concurrency | Manual locking | Automatic MVCC / locking |

**Interview Follow-up:**
- When would you still prefer a file system?
- What did we use before RDBMS in legacy banking systems?

**Short Interview Answer:**
A file system keeps unrelated files and leaves consistency and relationships to the developer, whereas a DBMS organizes data into related tables and manages security, consistency, concurrency and recovery for us.

---

## 3. What are the advantages of DBMS?

**Answer:**
Main advantages are less data redundancy, better consistency, controlled sharing, strong security, backup and recovery, and multi-user concurrent access. It also enforces integrity constraints so bad data can't be stored. In short, it replaces error-prone manual data handling with rules enforced by the engine.

**Example:**
In a banking system, a `CHECK (balance >= 0)` constraint plus a transaction guarantees you can't withdraw more than you have.

**Interview Follow-up:**
- Which advantage matters most in a banking system? (Consistency + integrity)
- What is the cost of these advantages? (See question 4)

**Short Interview Answer:**
DBMS gives us reduced redundancy, data consistency, security, controlled sharing, concurrency and easy backup/recovery compared to plain files.

---

## 4. What are the disadvantages of DBMS?

**Answer:**
The main drawbacks are cost (licenses and hardware), added complexity, and a performance overhead compared to raw file access. It also needs skilled DBAs, and the DBMS can become a single point of failure or a bottleneck. Schema changes can be painful in a large system.

**Interview Follow-up:**
- How do you reduce these problems? (Replication, caching, connection pooling, indexing)
- When is a NoSQL store preferred? (When schema is flexible and write throughput matters more)

**Short Interview Answer:**
DBMS costs more, needs administration, adds complexity and has some performance overhead — but for consistent, shared, critical data it is still worth it.

---

## 5. What is data abstraction?

**Answer:**
Data abstraction means hiding the internal complexity of how data is actually stored from the user. You work with tables and SQL, while the DBMS deals with files, blocks, indexes and disk I/O internally. That's why the same app can be moved from one DBMS to another with very little change.

**Example:**
A developer writes `SELECT * FROM orders` without caring whether orders are stored in a heap, a B+ tree file, or spread across partitions.

**Interview Follow-up:**
- How many levels of abstraction exist?
- Which level does JDBC sit at?

**Short Interview Answer:**
Data abstraction is the technique of hiding the physical storage details of data and exposing only a simple logical view to the user.

---

## 6. What are the levels of data abstraction?

**Answer:**
There are three levels: view (user), logical (conceptual) and physical. View level is what each user sees, logical level defines the overall structure and relationships, and physical level defines how data is actually stored on disk. Each level hides the details of the level below it.

| Level | What it defines | Who sees it |
|---|---|---|
| View / User | What each user sees | End users, reports |
| Logical / Conceptual | Tables, keys, relationships, constraints | DB designers, admins |
| Physical | Files, indexes, blocks, storage | DBMS internals |

**Interview Follow-up:**
- Which level gives data independence? (Changing one level without affecting the other)
- Give an example of logical data independence. (Adding a new column)

**Short Interview Answer:**
Data abstraction has three levels — view, logical and physical — and each level hides the details of the one below it.

---

## 7. What is data independence?

**Answer:**
Data independence means you can change the schema at one level without changing the schema at the level above it. In other words, the application code stays the same even if the storage design changes. That's why it is considered a major benefit of the DBMS.

**Example:**
If I add an extra column for internal caching, or split one big table into two, the Java code that does `SELECT name, email FROM users` doesn't need any modification.

**Interview Follow-up:**
- Name the two types. (Logical and physical data independence)
- Which one is harder to achieve? (Physical)

**Short Interview Answer:**
Data independence is the ability to modify the schema at one level of the DBMS without affecting the data consistency at the higher level.

---

## 8. What are the types of data independence?

**Answer:**
There are two: logical and physical. Logical independence means changing the logical schema (adding or removing attributes/tables) without affecting application programs. Physical independence means changing the internal storage — files, indexes, partitioning — without changing the logical schema.

| Type | What changes | What stays same |
|---|---|---|
| Logical | Tables, attributes, relationships | Application code |
| Physical | Files, indexes, storage, partitioning | Logical schema + app code |

**Example:**
Splitting `customer(name, city)` into `customer(id, name)` + `city(id, city)` is logical independence; adding an index on `city` is physical independence.

**Interview Follow-up:**
- How do you achieve physical independence? (Through views and mappings in the DBMS)
- Is Hibernate's ORM helping here? (Yes — it maps objects to tables, adding a layer of independence)

**Short Interview Answer:**
Logical data independence changes the schema without touching the app, while physical data independence changes storage without touching the schema.

---

## 9. What is a data model?

**Answer:**
A data model is a set of concepts and rules that describe how data is organised, related and constrained. It gives a common language to designers and developers for modelling a real-world system. For example, the ER model describes entities and relationships, the relational model describes tables and keys.

**Example:**
For an e-commerce app: Customer, Product, Order are entities; Customer is related to Order through a "places" relationship.

**Interview Follow-up:**
- Which model does a database designer use first? (Conceptual / ER model)
- What is the difference between a data model and a database? (Model is a blueprint; database is the actual data)

**Short Interview Answer:**
A data model is the blueprint that describes how data is structured, related and constrained — for example, the ER model or the relational model.

---

## 10. What are the types of data models?

**Answer:**
There are three types: conceptual, logical and physical. Conceptual is the high-level business view (entities, attributes, relationships). Logical is the implementation view in terms of tables, columns and keys. Physical is the storage view — files, indexes, access paths.

**Example:**
Conceptual: `Employee has skills`. Logical: `EMP_SKILL(emp_id, skill_id)`. Physical: `EMP_SKILL stored in a B+ tree with a hash index on emp_id`.

**Interview Follow-up:**
- Which model is storage-independent? (Both conceptual and logical)
- What does ORM do? (Maps the logical model to objects)

**Short Interview Answer:**
Data models are conceptual (entities/relationships), logical (tables/keys), and physical (files and indexes) — each is a more detailed view of the previous one.

---

## 11. What is schema and instance?

**Answer:**
Schema is the structure/blueprint of the database — table names, columns, data types, constraints. Instance is the current set of actual data stored in that structure at a given moment. Schema rarely changes; instance changes with every insert, update and delete.

**Example:**
Schema: `orders(id INT PRIMARY KEY, amount DECIMAL)`. Instance: the 5,000 rows currently sitting in that table.

**Interview Follow-up:**
- Which one is stored in the data dictionary? (Schema)
- Also known as? (Schema = metadata; instance = current snapshot of data)

**Short Interview Answer:**
Schema is the structural description of the database, and instance is the actual data present in it at any point in time.

---

## 12. What is the difference between logical schema and physical schema?

**Answer:**
The logical schema describes what data exists — tables, columns, keys, relationships, constraints — independent of storage. The physical schema describes how that data is stored — files, indexes, clusters, partitioning, compression. Logical schema is what developers see; physical schema is what the DB engine manages.

**Example:**
Logical: `users(id, email, name)`. Physical: rows stored in an InnoDB table with a clustered index on `id` and a secondary index on `email`.

**Interview Follow-up:**
- Who decides the physical schema? (DBA)
- Changing physical design should not break apps — why? (Physical data independence)

**Short Interview Answer:**
Logical schema is the logical design (tables and keys); physical schema is the storage design (files and indexes).

---

## 13. What is metadata?

**Answer:**
Metadata means "data about data" — information that describes the database itself. Table names, column types, constraints, indexes, privileges and schema version are all metadata. The DBMS stores this in its data dictionary so it can validate queries and enforce rules.

**Example:**
`information_schema` in MySQL stores metadata about every table, column and constraint in the server.

**Interview Follow-up:**
- Where is metadata stored? (Data dictionary / system catalog)
- Give a real-world example. (Flyway/Liquibase migration history table, JSON schema)

**Short Interview Answer:**
Metadata is data that describes other data — like table structures, column types and indexes — stored in the DBMS's data dictionary.

---

## 14. What is a data dictionary or catalog?

**Answer:**
A data dictionary is a centralized store of all metadata about a database — tables, columns, types, constraints, indexes and user privileges. The DBMS queries it whenever you run a statement, so it knows how to resolve `orders` and validate your columns. Catalog is used synonymously for this.

**Example:**
Before running `SELECT email FROM users`, MySQL reads the catalog to confirm `email` exists in `users` and what its type is.

**Interview Follow-up:**
- Where is it stored? (System tables in the DBMS's own storage)
- Does the application use it? (Yes — via `information_schema`, JDBC metadata, or Hibernate's schema validation)

**Short Interview Answer:**
A data dictionary is the DBMS's internal catalog of metadata — it stores the structure of the database so the engine can validate and run queries correctly.

---

## 15. What is an entity in DBMS?

**Answer:**
An entity is a real-world object or concept that we want to store data about — like Customer, Product or Order. In an ER diagram it's drawn as a rectangle. A good entity has a clear identity and its own attributes.

**Example:**
In a shopping app, `Customer` is an entity, and `name`, `email`, `phone` are its attributes.

**Interview Follow-up:**
- Entity vs attribute? (Entity = the object; attribute = its property)
- Strong or weak entity? (See question 27)

**Short Interview Answer:**
An entity is a real-world object we want to store information about, and it becomes a table in the relational model.

---

## 16. What are the types of entities?

**Answer:**
Common classifications are strong vs weak entities, regular vs key entities, and recursive vs binary/ternary entities. Strong entities have their own primary key; weak entities depend on a parent entity. Recursive entities participate in a relationship with themselves.

**Example:**
`Employee` is a strong entity with `emp_id`. `Employee_Dependents` may be a weak entity that only makes sense with an `emp_id` as part of its key.

**Interview Follow-up:**
- Can a weak entity exist alone? (No)
- What is a key entity? (An entity whose primary key is a single simple attribute, not composite)

**Short Interview Answer:**
Entities can be strong (own PK), weak (depend on an owner entity), key (single-attribute PK), or recursive (related to itself).

---

## 17. What is an attribute?

**Answer:**
An attribute is a property or characteristic of an entity. In the relational model, attributes become columns. Attributes can be simple or composite, single-valued or multi-valued, stored or derived, and one of them (or a set) becomes the key.

**Example:**
For a `Customer` entity: `name` (composite: first, last), `phone` (multi-valued), `age` (derived from DOB).

**Interview Follow-up:**
- Which attribute becomes the primary key? (The key attribute)
- Attribute vs column? (Same idea, different model layer)

**Short Interview Answer:**
An attribute is a property of an entity, and it becomes a column in the table when we convert the ER model to relations.

---

## 18. What are the types of attributes?

**Answer:**
Attributes are classified as simple or composite, single-valued or multi-valued, stored or derived, and key or non-key. Composite splits further into parts; multi-valued holds many values; derived is calculated, not stored.

| Type | Meaning | Example |
|---|---|---|
| Simple / Atomic | Can't be divided further | `age` |
| Composite | Made of sub-parts | `name` → first, last |
| Single-valued | One value only | `email` |
| Multi-valued | Many values | `phone_numbers` |
| Stored | Physically stored | `dob` |
| Derived | Calculated on the fly | `age` from `dob` |
| Key | Uniquely identifies the row | `customer_id` |
| Complex | Composite + multi-valued | `address(line1, line2, ...)` |

**Interview Follow-up:**
- Should derived attributes be stored? (Usually no, compute them)
- How are multi-valued attributes mapped to relations? (Separate table with FK)

**Short Interview Answer:**
Attributes can be simple, composite, single/multi-valued, stored or derived, key or non-key — the type tells me how to store them in a table.

---

## 19. What is a relationship?

**Answer:**
A relationship is an association between two or more entities. It's drawn as a diamond in an ER diagram and becomes a foreign key (or a separate join table) in the relational model. Relationships can have cardinality (1:1, 1:N, M:N) and can carry their own attributes.

**Example:**
Customer **places** Order. The "places" relationship has an attribute `order_date`.

**Interview Follow-up:**
- How do you map M:N to tables? (Create a junction table with both FKs)
- Can a relationship have attributes? (Yes, and they go in the junction table)

**Short Interview Answer:**
A relationship shows how entities are associated, and in the relational model it turns into foreign keys or a join table.

---

## 20. What are the types of relationships in DBMS?

**Answer:**
By degree: unary (recursive), binary, and ternary. By cardinality: one-to-one, one-to-many, and many-to-many. By participation: total vs partial, and identifying vs non-identifying.

**Example:**
`Employee` **manages** `Employee` is unary. `Customer` 1—N `Order` is one-to-many. `Student` M—N `Course` needs a `student_course` table.

**Interview Follow-up:**
- Which cardinality needs a separate table? (Many-to-many)
- Give a real 1:1 example. (A user profile table with `user_id` as both PK and FK)

**Short Interview Answer:**
Relationships can be unary, binary or ternary, and by cardinality they are 1:1, 1:N or M:N — M:N always becomes a separate junction table.

---

## 21. What are keys in DBMS?

**Answer:**
Keys are columns or sets of columns used to identify rows and to establish relationships between tables. There's a whole hierarchy: super key → candidate key → primary key, plus alternate, composite and foreign keys. A good key is unique, minimal and never changes.

**Example:**
`ORDER(order_id)` is the primary key, and `order_item(order_id, product_id)` has a composite key of both columns plus `order_id` as a foreign key.

**Interview Follow-up:**
- Which key is chosen by the developer? (Primary key)
- Can a table have multiple foreign keys? (Yes)

**Short Interview Answer:**
Keys identify records uniquely — super key, candidate key and primary key identify rows, while foreign keys connect tables together.

---

## 22. What is a super key?

**Answer:**
A super key is any set of one or more attributes that uniquely identifies every row in a relation — it just doesn't have to be minimal. If it contains extra unnecessary columns, it's still a valid super key, but not a candidate key.

**Example:**
For `EMP(emp_id, ssn, name)`, both `ssn` and `(emp_id, name)` are super keys because each uniquely identifies a row.

**Interview Follow-up:**
- What's the difference from a candidate key? (Minimality)
- Can a table have multiple super keys? (Yes, always)

**Short Interview Answer:**
A super key is any attribute set that uniquely identifies rows, even if it contains extra columns that aren't needed.

---

## 23. What is a candidate key?

**Answer:**
A candidate key is a minimal super key — the smallest set of attributes that still uniquely identifies each row, with no unnecessary attributes. Every table can have one or more candidate keys, and from those we pick the primary key.

**Example:**
`EMP(emp_id, ssn, name)` — `emp_id` and `ssn` are both candidate keys, so either could be the primary key.

**Interview Follow-up:**
- What if a candidate key has multiple columns? (It's a composite candidate key)
- Can a candidate key contain nulls? (No)

**Short Interview Answer:**
A candidate key is a minimal super key — the least set of attributes that uniquely identifies each row.

---

## 24. What is a primary key?

**Answer:**
A primary key is the candidate key we choose to uniquely identify each row in a table. It can never be null and must be unique, and the DBMS enforces both automatically. It's the main way rows are referenced by other tables.

**Example:**
`CREATE TABLE users (id BIGINT PRIMARY KEY AUTO_INCREMENT, email VARCHAR(255) UNIQUE);` — `id` is the PK and `email` becomes an alternate key.

**Interview Follow-up:**
- Can a table have two primary keys? (No, only one — but it can be composite)
- Why use auto-increment instead of UUID? (Smaller, index-friendly; UUID is better for distributed systems)
- Why not use natural keys? (They can change and are hard to keep unique)

**Short Interview Answer:**
A primary key is the chosen unique, non-null identifier for each row in a table, and other tables reference it via foreign keys.

---

## 25. What is a foreign key?

**Answer:**
A foreign key is a column in one table that references the primary key of another table. It creates a relationship between tables and prevents orphan rows by ensuring the referenced value exists. Violating it raises a referential integrity error.

**Example:**
`orders(customer_id BIGINT, FOREIGN KEY (customer_id) REFERENCES customers(id))` — an order can't exist without a valid customer.

**Interview Follow-up:**
- What are the actions on delete/update? (`ON DELETE CASCADE`, `SET NULL`, `RESTRICT`, `NO ACTION`)
- Is a FK mandatory? (No — some high-performance systems skip them and enforce it in the app, but that's risky)

**Short Interview Answer:**
A foreign key links two tables by referencing the primary key of another table, and it maintains referential integrity between them.

## 26. What is a composite key?

**Answer:**
A composite key is a key made of two or more columns together, where only that combination is unique. A single column inside it may repeat many times — for example in a many-to-many junction table.

**Example:**
In `student_course(student_id, course_id)`, each student takes many courses and each course has many students, so only `(student_id, course_id)` together is unique.

**Interview Follow-up:**
- Why not use a surrogate key? (You can, but a composite key expresses the real rule better)
- Does a composite key need its own index? (Yes, and the DBMS needs a UNIQUE constraint on all columns)

**Short Interview Answer:**
A composite key is a primary or unique key built from two or more columns combined, which is common in many-to-many junction tables.

---

## 27. What is a weak entity?

**Answer:**
A weak entity has no key of its own — it can't be identified independently, so it depends on an owner (identifying) entity. It's converted into a table whose primary key includes the owner's primary key as a foreign key. This is called an existence dependency.

**Example:**
`Dependent(emp_id, dep_name)` — a dependent only makes sense together with an employee, so `(emp_id, dep_name)` becomes the primary key.

**Interview Follow-up:**
- How is it drawn in ER? (Double rectangle, double diamond)
- Weak vs strong entity? (Weak has no PK on its own)

**Short Interview Answer:**
A weak entity is one that has no primary key of its own and depends on an owner entity, so its table's primary key includes the owner's foreign key.

---

## 28. What is generalization and specialization?

**Answer:**
These are the same concept viewed from opposite directions. Specialization is top-down: you take a general entity and split it into more specific sub-entities based on their special properties. Generalization is bottom-up: you combine multiple entities that share common features into a single higher-level entity.

**Example:**
Specialization: `Employee` → `Manager` and `Engineer`, where only `Manager` has an extra `department` attribute. Generalization: `Car` + `Bike` → `Vehicle` with common attributes like `color` and `price`.

**Interview Follow-up:**
- How is it implemented in relational model? (Single table with discriminator, or table per subtype with a FK to parent)
- Which is better? (Depends — single table is simple, table-per-type is better when subtypes have very different columns)

**Short Interview Answer:**
Specialization splits a general entity into sub-entities from top down, and generalization merges common entities into a super-entity from bottom up.

---

## 29. What is aggregation in ER model?

**Answer:**
Aggregation means treating a relationship between entities as if it were a single entity, so that other entities can relate to that relationship. It's used for relationships that have their own attributes and can participate in further relationships, like M:N.

**Example:**
In a university, `Student` and `Course` have an M:N `enrollment` relationship. That enrollment is graded, so we treat enrollment as an entity and let `Grading` relate to it.

**Interview Follow-up:**
- How do you map aggregation to tables? (The relationship becomes its own table with both FKs plus its attributes)
- Difference between aggregation and composition? (Aggregation is a "has-a" whole-part without ownership; composition is exclusive ownership)

**Short Interview Answer:**
Aggregation lets you treat a relationship as an entity itself so other entities can link to it, which is how we handle M:N relationships with their own attributes.

---

## 30. How do you convert an ER diagram into a relational schema?

**Answer:**
The steps are: convert each strong entity to a table, convert weak entities with an FK to the owner, convert 1:1 and 1:N relationships into foreign keys, and convert M:N relationships into a separate junction table. Keys, relationship attributes and multivalued attributes are placed appropriately, then you check the normalisation form.

**Example:**
`Customer — places → Order (1:N)` becomes `orders(customer_id FK)`. `Student — enrolls → Course (M:N)` becomes `enrollment(student_id FK, course_id FK)`.

**Interview Follow-up:**
- Where does the relationship attribute go? (In the FK table for 1:N, in the junction table for M:N)
- What about multivalued attributes? (Separate table with FK back to the entity)

**Short Interview Answer:**
Each entity becomes a table, 1:N and 1:1 relationships become foreign keys, M:N relationships become a junction table, and weak entities combine the owner's PK into their own.

---

## 31. What is a relation in DBMS?

**Answer:**
A relation is a table — a set of rows (tuples) with the same attributes (columns). In the relational model, a relation is formally a set of tuples over a fixed schema, and it must satisfy certain properties. In practice we call it a table and rows can technically repeat, which is why real DBMSs relax the theory slightly.

**Example:**
`EMP(emp_id, name, dept_id)` is a relation; `(101, 'Ravi', 5)` is one tuple in it.

**Interview Follow-up:**
- Relation vs table? (Same idea; tables allow duplicate rows and ordering, mathematical relations don't)

**Short Interview Answer:**
A relation is a table in the relational model — a set of tuples that all share the same set of attributes.

---

## 32. What are the properties of a relation?

**Answer:**
A relation has: unique attribute names, a single homogeneous domain per attribute, atomic (indivisible) values, no duplicate tuples, and a well-defined primary key. The order of rows and columns doesn't matter mathematically.

**Example:**
Storing `"Delhi, 110001"` in one `address` column breaks atomicity; you should split it into `city` and `pincode`.

**Interview Follow-up:**
- Which property is violated most often? (Atomicity — stuffing lists or comma-separated values into one column)
- How do you fix atomicity issues? (Normalize — split into separate tables)

**Short Interview Answer:**
A relation must have unique column names, atomic values, one domain per attribute, and no duplicate rows, with a key to identify each tuple.

---

## 33. What are tuples and attributes?

**Answer:**
A tuple is a row of data (a record), and an attribute is a column (a field of that record). Together a tuple describes one complete instance of the entity. For `EMP(101, 'Ravi', 5)`, `101`, `'Ravi'` and `5` are attribute values of that tuple.

**Interview Follow-up:**
- Tuple vs record vs row? (Same concept, different terminology)
- What is a domain? (The set of allowed values for an attribute)

**Short Interview Answer:**
A tuple is a row representing one record, and an attribute is a column representing one property of that record.

---

## 34. What are integrity constraints?

**Answer:**
Integrity constraints are rules that ensure the data in the database stays correct and consistent. They can be column-level (NOT NULL, DEFAULT, CHECK) or table-level (PRIMARY KEY, UNIQUE, FOREIGN KEY). The DBMS rejects any statement that breaks them, so bad data never gets stored.

**Example:**
`CHECK (age >= 0)` stops someone from inserting `age = -5`, and a FK stops an `orders` row pointing to a customer that doesn't exist.

**Interview Follow-up:**
- Domain vs key vs referential integrity? (Values in range / row identity / links to other tables)
- Entity integrity? (Primary keys are never null)

**Short Interview Answer:**
Integrity constraints are validation rules like NOT NULL, UNIQUE, PRIMARY KEY, FOREIGN KEY and CHECK that keep the data correct and consistent.

---

## 35. What are relational constraints?

**Answer:**
Relational constraints are the constraints defined on a relation — domain, key, entity integrity and referential integrity. They're a subset of integrity constraints that are specifically about the relational model. Domain constraints check the allowed values of an attribute, while key constraints guarantee unique identification.

**Example:**
`age INT CHECK (age BETWEEN 0 AND 150)` is a domain constraint; `FOREIGN KEY (dept_id) REFERENCES dept(id)` is a referential constraint.

**Interview Follow-up:**
- Name the four relational constraints. (Domain, key, entity integrity, referential integrity)
- Are these enforced by the DBMS or app? (Prefer DBMS)

**Short Interview Answer:**
Relational constraints are rules on a relation — domain, key, entity and referential integrity — that keep rows valid and properly linked.

---

## 36. What is SQL?

**Answer:**
SQL (Structured Query Language) is the standard language used to communicate with a relational database. It lets you define, query, update and control data, and it's declarative — you tell the DBMS what you want, not how to get it. It's used by JDBC, Hibernate and every backend framework.

**Example:**
`SELECT name, SUM(amount) FROM orders GROUP BY name HAVING SUM(amount) > 1000;`

**Interview Follow-up:**
- How many dialects exist? (MySQL, PostgreSQL, Oracle, T-SQL, SQLite)
- Is SQL case-sensitive? (Keywords are not; string values may be, depending on DBMS and collation)

**Short Interview Answer:**
SQL is the standard declarative language to create, read, update and delete relational data, and it's what a backend developer uses daily through JDBC or an ORM.

---

## 37. What are DDL commands?

**Answer:**
DDL (Data Definition Language) commands define and change the structure of the database. The main ones are `CREATE`, `ALTER`, `DROP`, `TRUNCATE` and `RENAME`. In practice these auto-commit and can't be rolled back inside a transaction.

**Example:**
`CREATE TABLE orders (id BIGINT PRIMARY KEY, amount DECIMAL(10,2));`

**Interview Follow-up:**
- Why can't DDL be rolled back? (It performs implicit commit in most DBMSs)
- Who should run DDL in production? (A migration tool like Flyway/Liquibase, not manually)

**Short Interview Answer:**
DDL commands define the database structure — CREATE, ALTER, DROP, TRUNCATE — and they implicitly commit in most databases.

---

## 38. What are DML commands?

**Answer:**
DML (Data Manipulation Language) commands deal with the data inside tables: `SELECT`, `INSERT`, `UPDATE`, `DELETE` and `MERGE`. These are the operations that change rows and can be rolled back inside a transaction. This is the category used most often in applications.

**Example:**
`UPDATE accounts SET balance = balance - 500 WHERE id = 1;`

**Interview Follow-up:**
- Is SELECT DML? (Yes, though it doesn't modify data)
- Difference from DDL? (DML changes rows, DDL changes structure)

**Short Interview Answer:**
DML commands manipulate the data — SELECT, INSERT, UPDATE and DELETE — and unlike DDL they are fully transaction-controlled.

---

## 39. What are DCL commands?

**Answer:**
DCL (Data Control Language) commands manage access and permissions. `GRANT` gives rights to a user or role, and `REVOKE` takes them back. In practice most teams use roles (`app_user`, `readonly`) instead of granting to individual users.

**Example:**
`GRANT SELECT, INSERT ON orders TO app_user;`

**Interview Follow-up:**
- Name the privilege levels. (Database, table, column, role-level)
- Where are these used? (Least-privilege principle — app should never use DBA credentials)

**Short Interview Answer:**
DCL commands are GRANT and REVOKE, and they control who can read or modify the database.

---

## 40. What are TCL commands?

**Answer:**
TCL (Transaction Control Language) commands control transactions. They are `COMMIT`, `ROLLBACK`, `SAVEPOINT` and `START TRANSACTION`. They decide whether the changes inside a transaction are permanently saved or undone.

**Example:**
```sql
START TRANSACTION;
UPDATE account SET balance = balance - 500 WHERE id = 1;
SAVEPOINT after_debit;
UPDATE account SET balance = balance + 500 WHERE id = 2;
ROLLBACK TO after_debit;  -- only the credit is undone
COMMIT;
```

**Interview Follow-up:**
- Why use savepoints? (To undo part of a transaction)
- What is Spring's `@Transactional`? (It wraps the method in commit/rollback based on exceptions)

**Short Interview Answer:**
TCL commands — COMMIT, ROLLBACK and SAVEPOINT — control transaction boundaries and decide if changes are saved or undone.

---

## 41. What is the difference between DELETE, TRUNCATE, and DROP?

**Answer:**
`DELETE` removes rows one by one using a transaction and can be filtered with `WHERE`, so it is slower and fully recoverable. `TRUNCATE` deletes all rows in one shot, resets the identity counter and is faster, but can't be filtered or rolled back. `DROP` removes the entire table structure along with its data.

**Answer Table:**

| Property | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| Removes | Specific rows | All rows | Table + data |
| WHERE clause | Yes | No | No |
| Transaction/rollback | Yes | No | No |
| Speed | Slow (row-by-row) | Fast | Fast |
| Resets identity/auto-increment | No | Yes | Irrelevant |
| Structure kept | Yes | Yes | No |

**Interview Follow-up:**
- Why is DELETE slower? (Every row deletion is logged individually)
- What about TRUNCATE in PostgreSQL? (It's transactional there)

**Short Interview Answer:**
DELETE removes specific rows with a WHERE clause and can be rolled back, TRUNCATE wipes all rows fast and resets identity but can't be rolled back, and DROP removes the whole table.

---

## 42. What are SQL constraints?

**Answer:**
SQL constraints are rules defined on a table that the DBMS always enforces. The main ones are `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK` and `DEFAULT`. They protect data integrity even if multiple applications write to the same table.

**Example:**
```sql
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(255) NOT NULL UNIQUE,
  age INT CHECK (age >= 0),
  status VARCHAR(20) DEFAULT 'ACTIVE'
);
```

**Interview Follow-up:**
- Can you drop a constraint later? (Yes, with `ALTER TABLE ... DROP CONSTRAINT`)
- Why prefer DB constraints over validation in code? (Multiple apps share the DB; the DBMS is the last line of defence)

**Short Interview Answer:**
SQL constraints are rules like NOT NULL, UNIQUE, PRIMARY KEY, FOREIGN KEY and CHECK that the database always enforces to keep data valid.

---

## 43. What are joins in SQL?

**Answer:**
A join combines rows from two or more tables based on a related column. Usually you join a foreign key to a primary key to get meaningful, denormalized data for display. The result is logically one table, built on the fly.

**Example:**
```sql
SELECT o.id, c.name, o.amount
FROM orders o
JOIN customers c ON c.id = o.customer_id;
```

**Interview Follow-up:**
- Why avoid `SELECT *` with joins? (Unnecessary I/O and unreadable results)
- What if I need only unmatched rows? (LEFT JOIN)

**Short Interview Answer:**
A join merges rows from multiple tables using a common column — typically matching a foreign key with a primary key — so I can query related data in one statement.

---

## 44. What are the types of joins?

**Answer:**
The main ones are INNER, LEFT, RIGHT, FULL OUTER, CROSS and SELF joins. INNER returns only matching rows, LEFT returns all left rows plus matches, RIGHT is the mirror of LEFT, and FULL returns all rows from both sides. They are usually written with `INNER JOIN` and `OUTER JOIN` keywords.

**Example:**

| Type | Returns |
|---|---|
| INNER JOIN | Only rows that match in both tables |
| LEFT JOIN | All rows of the left table + matching right rows (NULL if no match) |
| RIGHT JOIN | All rows of the right table + matching left rows |
| FULL OUTER JOIN | All rows from both tables, NULLs where unmatched |
| CROSS JOIN | Every row × every row (Cartesian product) |
| SELF JOIN | A table joined with itself (e.g., employee–manager) |

```sql
SELECT c.name, o.amount
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id;   -- customers with no orders also appear
```

**Interview Follow-up:**
- MySQL-specific? (MySQL has no FULL OUTER JOIN — emulate with `LEFT JOIN UNION RIGHT JOIN`)
- What is an implicit join? (Old-style comma-separated FROM, and cross join without ON)

**Short Interview Answer:**
Joins combine tables on a shared column — INNER for matches only, LEFT/RIGHT for one-sided matches, FULL for everything, plus CROSS and SELF for special cases.

---

## 45. What is a subquery?

**Answer:**
A subquery is a SELECT query nested inside another query, used in the WHERE, FROM, SELECT or HAVING clause. It lets you filter or compute using the result of another query, often to avoid complex joins. For readability, some teams prefer CTEs.

**Example:**
```sql
SELECT name FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**Interview Follow-up:**
- Subquery vs CTE? (CTE is named, more readable, and can be recursive)
- IN vs EXISTS? (`EXISTS` is usually faster because it stops at the first match)

**Short Interview Answer:**
A subquery is a query nested inside another query, used mainly in the WHERE clause to filter results based on another query's output.

---

## 46. What is a correlated subquery?

**Answer:**
A correlated subquery references a column from the outer query, so it must re-run for every row of the outer query. Because of that it can be slow on large tables. Modern DBMS optimizers often rewrite it into a join internally.

**Example:**
```sql
SELECT e.name
FROM employees e
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.emp_id = e.id);
```

**Interview Follow-up:**
- How to improve performance? (Add an index on the correlated column, or rewrite as a JOIN/CTE)
- Correlated vs uncorrelated? (Correlated depends on the outer query and runs per row)

**Short Interview Answer:**
A correlated subquery refers to the outer query's columns, so it's evaluated once per row of the outer query.

---

## 47. What is a view in SQL?

**Answer:**
A view is a virtual table defined by a `SELECT` query — it stores the query, not the data. It can be used like a table to simplify complex queries, hide sensitive columns, or present a stable interface even when the base tables change. Updates through a simple view are possible in most DBMSs.

**Example:**
```sql
CREATE VIEW active_users AS
SELECT id, name FROM users WHERE status = 'ACTIVE';
```

**Interview Follow-up:**
- Materialized view? (It stores the result physically and must be refreshed — faster to read, used for reports)
- Is a view faster? (No — it adds a layer, unless materialized)

**Short Interview Answer:**
A view is a virtual table created from a query, used to simplify complex SQL, hide columns, or give a stable interface to the data.

---

## 48. What are indexes in DBMS?

**Answer:**
An index is a separate data structure (usually a B+ tree) that lets the DBMS find rows quickly instead of scanning the whole table. It speeds up SELECT, WHERE and JOIN operations dramatically, but it uses extra disk space and slows down INSERT/UPDATE/DELETE because every change must also update the index. So we create indexes selectively on columns used in filtering, joining and sorting.

**Example:**
`CREATE INDEX idx_orders_customer ON orders(customer_id);` — turns a full scan of 10 million rows into a small index lookup.

**Interview Follow-up:**
- What is a composite index? (An index on multiple columns; order matters — leftmost prefix is used)
- Why does an index sometimes not get used? (Low selectivity, function on the column, leading wildcard `LIKE '%x'`)
- B+ tree over hash? (B+ tree supports range queries too; hash only supports equality)

**Short Interview Answer:**
An index is an auxiliary structure like a B+ tree that speeds up lookups, at the cost of extra space and slower writes.

---

## 49. What are stored procedures?

**Answer:**
A stored procedure is a named block of SQL code stored inside the database and executed by name. It can accept parameters, run multiple statements, and return results. The benefit is less network traffic and the logic stays close to the data; the downside is that business logic in the DB becomes hard to version and test.

**Example:**
```sql
CREATE PROCEDURE transfer(amt DECIMAL, fromId INT, toId INT)
BEGIN
  UPDATE accounts SET balance = balance - amt WHERE id = fromId;
  UPDATE accounts SET balance = balance + amt WHERE id = toId;
END;
```

**Interview Follow-up:**
- Procedure vs function? (Functions return a value, procedures can modify data)
- Why are they less popular now? (Migration, testing and versioning difficulties; most teams keep logic in the app)

**Short Interview Answer:**
A stored procedure is precompiled SQL code stored and executed inside the database, reducing network calls but moving business logic away from the application.

---

## 50. What are triggers?

**Answer:**
A trigger is a piece of SQL that fires automatically when a specific event happens on a table — INSERT, UPDATE or DELETE. It's used for audit logging, maintaining summary tables, or enforcing business rules. The thing to remember is that triggers run invisibly in the background, so their side effects can surprise you.

**Example:**
```sql
CREATE TRIGGER audit_orders AFTER INSERT ON orders
FOR EACH ROW INSERT INTO order_log(order_id, action, at)
VALUES (NEW.id, 'CREATED', NOW());
```

**Interview Follow-up:**
- Before vs AFTER trigger? (BEFORE can modify the new row; AFTER runs post-change)
- Why avoid triggers in OLTP? (Hidden side effects, hard debugging, extra latency)

**Short Interview Answer:**
A trigger automatically runs SQL code on INSERT, UPDATE or DELETE — commonly used for audit logs and keeping derived tables in sync.

## 51. What is normalization?

**Answer:**
Normalization is the process of organizing tables to reduce data redundancy and dependency problems. You split a wide table into related smaller tables based on functional dependencies, so each fact is stored only once. The forms are 1NF, 2NF, 3NF, BCNF and beyond.

**Example:**
Splitting `EMP(emp_id, emp_name, dept_id, dept_name, dept_location)` into `EMP(emp_id, emp_name, dept_id)` and `DEPT(dept_id, dept_name, dept_location)` removes repeated department data.

**Interview Follow-up:**
- Up to which normal form should you go? (Usually 3NF for OLTP; denormalize for read-heavy systems)
- Is normalization always good? (No — for OLAP/reporting, denormalization often performs better)

**Short Interview Answer:**
Normalization organizes data into tables by functional dependencies to remove redundancy and anomalies, going step by step through 1NF, 2NF, 3NF and BCNF.

---

## 52. Why is normalization needed?

**Answer:**
Normalization removes redundancy, so data is stored in one place and stays consistent. Without it you get insert, update and delete anomalies, where one logical fact is spread across multiple rows. It also makes the schema easier to understand and maintain.

**Example:**
If department name is stored in every employee row, renaming a department requires updating 1000 rows, and missing one leaves contradictory data.

**Interview Follow-up:**
- What is the trade-off? (Extra joins and slightly slower reads)
- Where do you denormalize deliberately? (Read-heavy dashboards, search indexes, data warehouses)

**Short Interview Answer:**
Normalization is needed to eliminate redundancy and update/insert/delete anomalies by storing each fact in exactly one place.

---

## 53. What are anomalies in DBMS?

**Answer:**
Anomalies are problems that appear when a table is poorly designed. There are three: insert anomaly (can't add data without unrelated data), update anomaly (same fact stored in multiple rows), and delete anomaly (deleting one row accidentally loses other data).

**Example:**
In `EMP(emp_id, emp_name, dept_name)` without a separate DEPT table, you can't insert a new empty department, renaming a department needs many updates, and deleting the last employee of a department deletes the department info.

**Interview Follow-up:**
- Which normal form fixes update anomaly? (2NF/3NF by removing partial/transitive dependencies)
- Which anomaly is most commonly asked about? (Update anomaly)

**Short Interview Answer:**
Anomalies are inconsistencies from bad design — you can't insert properly, you must update the same data in many rows, and deleting a row can lose unrelated data.

---

## 54. What is functional dependency?

**Answer:**
A functional dependency `X → Y` means that if two rows have the same value of X, they must have the same value of Y — X determines Y. Candidate keys are determined by finding attributes that functionally determine all others.

**Example:**
`emp_id → emp_name, dept_id` means one employee ID always maps to the same name and department.

**Interview Follow-up:**
- Trivial vs non-trivial FD? (Trivial: Y is a subset of X, e.g. `(a,b) → a`)
- Why does it matter? (It's the mathematical basis for normalization)

**Short Interview Answer:**
A functional dependency `X → Y` means X uniquely determines Y, and it's the core concept used to decide how to split tables during normalization.

---

## 55. What is partial dependency?

**Answer:**
A partial dependency exists when part of a composite key determines a non-key attribute. This violates 2NF, because some columns depend only on half the key. The fix is to move that attribute into a table with that partial key.

**Example:**
In `EMP_SKILL(emp_id, skill_id, skill_name, emp_city)`, `skill_name` depends only on `skill_id` — a partial dependency.

**Interview Follow-up:**
- Does it matter only for composite keys? (Yes, it only exists for composite keys)
- Which NF removes it? (2NF)

**Short Interview Answer:**
A partial dependency is when part of a composite key determines another column, and it's removed by normalizing to 2NF.

---

## 56. What is transitive dependency?

**Answer:**
A transitive dependency is when a non-key attribute depends on another non-key attribute rather than directly on the key — that is, `key → X → Y`. This violates 3NF. The fix is to move Y into its own table with X as its key.

**Example:**
In `EMP(emp_id, dept_id, dept_name)`, `emp_id → dept_id → dept_name` means `dept_name` is transitively dependent, so it belongs in a DEPT table.

**Interview Follow-up:**
- Partial vs transitive? (Partial = part of composite key; transitive = through another non-key column)
- Which NF fixes it? (3NF)

**Short Interview Answer:**
A transitive dependency is when a non-key column depends on another non-key column instead of the key, and 3NF removes it.

---

## 57. What is First Normal Form (1NF)?

**Answer:**
1NF means every column holds a single atomic value, with no repeating groups or nested lists. If a column contains multiple values or a table has hidden lists like `phone1, phone2`, the table isn't in 1NF. It's the most basic requirement.

**Example:**
`customer(id, name, phones)` with `'999,888,777'` violates 1NF. Split it into `customer_phone(id, phone)`.

**Interview Follow-up:**
- Is splitting columns (first/last) required for 1NF? (Depends — only if you need to query them separately; strictly, the current value is atomic)
- What's the first step of normalization? (Convert to 1NF)

**Short Interview Answer:**
1NF requires every column to contain a single atomic value — no arrays, no comma-separated lists and no repeating groups.

---

## 58. What is Second Normal Form (2NF)?

**Answer:**
2NF means the table is in 1NF and has no partial dependency — every non-key attribute depends on the whole primary key. It only matters for tables with composite keys; tables with a single-column key are already in 2NF once they're in 1NF. The fix is to split the partially dependent attributes into a new table.

**Example:**
`EMP_SKILL(emp_id, skill_id, skill_name, emp_city)` — `emp_city` depends only on `emp_id`, so move it to `EMP`.

**Interview Follow-up:**
- Is 2NF possible without a composite key? (It's automatically satisfied)
- Common in practice? (Yes, very common in junction tables)

**Short Interview Answer:**
2NF means 1NF plus no partial dependency — every non-key column must depend on the entire composite primary key.

---

## 59. What is Third Normal Form (3NF)?

**Answer:**
3NF means 2NF plus no transitive dependency — non-key attributes must not depend on other non-key attributes. Formally, for every `X → Y`, X must be a superkey or Y must be a prime attribute. 3NF is the usual target for OLTP schemas.

**Example:**
`EMP(emp_id, dept_id, dept_name, dept_city)` violates 3NF; move `dept_name` and `dept_city` to `DEPT(dept_id, dept_name, dept_city)`.

**Interview Follow-up:**
- Difference between 3NF and BCNF? (3NF allows a non-superkey to determine a prime attribute; BCNF doesn't)
- Should every table be in 3NF? (For OLTP yes, for OLAP no)

**Short Interview Answer:**
3NF means 2NF plus no transitive dependency, so no non-key attribute depends on another non-key attribute.

---

## 60. What is BCNF?

**Answer:**
BCNF (Boyce-Codd Normal Form) is a stricter version of 3NF. It requires that for every non-trivial functional dependency `X → Y`, X must be a superkey. 3NF allows an exception where Y is a prime attribute; BCNF removes that exception entirely. BCNF handles most practical anomalies but not every dependency problem — 4NF and 5NF go further.

**Example:**
In `R(teacher, subject, student)` with dependencies `teacher → subject` and `(teacher, student) → ...`, the teacher→subject dependency violates BCNF because teacher isn't a superkey. You split it into `TEACHER_SUBJECT` and `STUDENT_TEACHER`.

**Interview Follow-up:**
- Does BCNF always guarantee lossless joins? (BCNF relations aren't always dependency preserving; 4NF/5NF target remaining issues)
- Is BCNF practical? (Yes for most OLTP designs, but a few extra tables)

**Short Interview Answer:**
BCNF is a stricter 3NF where every determinant must be a superkey, so it removes even the exceptions that 3NF allows.

---

## 61. What is denormalization?

**Answer:**
Denormalization is deliberately duplicating or pre-joining data after normalization to improve read performance. It reduces the number of joins at the cost of extra storage and redundancy. It's common in read-heavy systems like dashboards, search indexes and OLAP.

**Example:**
Storing `customer_name` directly in an `orders` table so listing orders doesn't require joining `customers` — useful for high-volume reads, risky if the name changes.

**Interview Follow-up:**
- When do you use it? (After measuring — when read performance matters more than consistency)
- How do you handle the inconsistency? (Sync via triggers, CDC, or async events)

**Short Interview Answer:**
Denormalization intentionally duplicates data to speed up reads and reduce joins, trading data consistency for performance.

---

## 62. What is a transaction in DBMS?

**Answer:**
A transaction is a logical unit of work made of one or more SQL statements that either all succeed or all fail. It gives you ACID guarantees, so a failure halfway through leaves no half-applied state. In Spring, you mark it with `@Transactional`.

**Example:**
A bank transfer is one transaction: debit account A and credit account B. If B fails, A must not stay debited.

**Interview Follow-up:**
- Why does Spring need `@Transactional` on public methods? (Spring AOP proxies don't intercept self-invocation)
- What happens on a runtime exception? (Rollback by default; checked exceptions need `rollbackFor`)

**Short Interview Answer:**
A transaction is a group of SQL statements treated as a single unit of work — either all of them commit or all get rolled back.

---

## 63. What are ACID properties?

**Answer:**
ACID is the set of guarantees a transaction provides: Atomicity (all or nothing), Consistency (constraints stay valid), Isolation (concurrent transactions don't see partial data), and Durability (committed data survives crashes). It's implemented using locking, WAL and recovery logs inside the DBMS.

**Example:**
For `transfer(500, A, B)`: if the server crashes after commit, the money is still transferred — that's durability; if it crashes midway, nothing is applied — that's atomicity.

**Interview Follow-up:**
- Which is hardest? (Isolation — it's the one with real trade-offs)
- What does isolation give you? (Serializability and consistent snapshots)

**Short Interview Answer:**
ACID stands for Atomicity, Consistency, Isolation and Durability — the four guarantees that make a transaction reliable.

---

## 64. What are the states of a transaction?

**Answer:**
A transaction moves through five states: active, partially committed, committed, failed, and aborted/terminated. After all statements execute successfully it becomes partially committed; `COMMIT` moves it to committed, and if an error occurs it moves to failed and then to aborted after rollback.

**Example:**
An online payment that passes validation, updates the order and the ledger, then fails at the last step — it goes from active → failed → aborted, and all changes are rolled back.

**Interview Follow-up:**
- Where does it go from committed? (Terminated, and a new transaction starts)
- What can cause a failed state? (Deadlock, constraint violation, timeout, system crash)

**Short Interview Answer:**
A transaction is active, partially committed, committed, failed or aborted — it rolls back only from the failed state.

---

## 65. What is concurrency control?

**Answer:**
Concurrency control makes sure multiple transactions running at the same time still produce correct results. The DBMS does it with locking (2PL), MVCC, and timestamp ordering, choosing between serializability and higher throughput. Without it, you'd get dirty reads, lost updates and inconsistent reports.

**Example:**
Two people buying the last seat — concurrency control ensures only one transaction gets it, the other gets a failure instead of both succeeding.

**Interview Follow-up:**
- Locking vs MVCC? (Locking blocks readers; MVCC gives readers a consistent snapshot without blocking)
- What's the isolation anomaly trade-off? (Higher isolation = more consistency, less concurrency)

**Short Interview Answer:**
Concurrency control ensures parallel transactions still behave as if they ran one after another, using locks, MVCC or timestamps.

---

## 66. What is a schedule?

**Answer:**
A schedule is the chronological order in which the operations of multiple transactions are interleaved on the CPU. It's represented as a sequence of read, write, commit and abort operations. Two schedules are equivalent if they produce the same final database state.

**Example:**
`T1: R(A), W(B)` interleaved with `T2: R(B), W(A)` — this schedule can produce an inconsistent state depending on order.

**Interview Follow-up:**
- What is a schedule of a single transaction? (It's a serial schedule by definition)
- Why do we study schedules? (To decide whether concurrent execution is safe)

**Short Interview Answer:**
A schedule is the time-ordered sequence of operations from one or more transactions running together.

---

## 67. What is serial and concurrent schedule?

**Answer:**
A serial schedule runs one transaction completely before starting the next — no interleaving, so it's always correct but slow. A concurrent (interleaved) schedule mixes operations of multiple transactions for better CPU and I/O utilisation. The goal is concurrency that is still correct, i.e., serializable.

**Example:**
Serial: T1 fully runs, then T2. Concurrent: T1 writes, T2 writes, T1 commits, T2 commits.

**Interview Follow-up:**
- Is serial always used? (No — it wastes CPU during I/O waits)
- What do we want instead? (A concurrent but serializable schedule)

**Short Interview Answer:**
A serial schedule runs transactions one after another and is always correct; a concurrent schedule interleaves them for better performance.

---

## 68. What is serializability?

**Answer:**
Serializability means a concurrent schedule is equivalent to some serial schedule — so the interleaved result is indistinguishable from running them one by one. This is the correctness criterion for concurrency control. It's the main thing we aim for in a transactional database.

**Example:**
Two transfers running at the same time, but if the result matches running them sequentially, the schedule is serializable.

**Interview Follow-up:**
- Two ways to test it? (Conflict serializability and view serializability)
- Does serializable guarantee no anomalies? (Yes — no dirty reads, lost updates, or non-repeatable reads)

**Short Interview Answer:**
A schedule is serializable if its result is the same as running the transactions one after another in some order.

---

## 69. What is conflict serializability?

**Answer:**
Conflict serializability means the schedule can be transformed into a serial schedule purely by swapping non-conflicting operations. Two operations conflict if they belong to different transactions, touch the same item, and at least one is a write. Practically you check it by building a precedence graph and testing for cycles — an acyclic graph means serializable.

**Example:**
`T1: W(A)`, `T2: R(A)` conflict and can't be swapped; `T1: R(A)`, `T2: R(B)` don't conflict and can be reordered freely.

**Interview Follow-up:**
- How is it checked? (Precedence graph, topological sort)
- Is it easy to verify? (Yes, much easier than view serializability — that's why it's preferred)

**Short Interview Answer:**
Conflict serializability means a schedule can be reordered into a serial one by swapping only non-conflicting operations, and it's checked using a precedence graph.

---

## 70. What is view serializability?

**Answer:**
View serializability means the schedule produces the same final reads as some serial schedule, even though the intermediate states may differ. It's a weaker condition than conflict serializability, and checking it requires solving NP-hard problems. Because of that, the industry uses conflict serializability instead.

**Example:**
A transaction that only reads can sometimes be reordered safely even though it shares writes with others — view serializability allows it.

**Interview Follow-up:**
- Why isn't it used in practice? (Too expensive to check, NP-hard)
- Which is stricter? (Conflict serializability is stricter)

**Short Interview Answer:**
View serializability matches the final result of a serial schedule even if intermediate states differ, but checking it is NP-hard, so conflict serializability is preferred.

---

## 71. What are concurrency problems in DBMS?

**Answer:**
The classic problems are dirty read, non-repeatable read, phantom read, lost update and inconsistent analysis. Each is a way two transactions can see or corrupt each other's data. Isolation levels exist specifically to control which of these problems are allowed.

**Example:**

| Problem | Meaning | Cause |
|---|---|---|
| Dirty read | Reading uncommitted data | T1 rolls back after T2 read its data |
| Non-repeatable read | Same row, different value on re-read | Another transaction updated & committed it |
| Phantom read | Extra rows appear on re-query | Another transaction inserted matching rows |
| Lost update | One update silently overwrites another | Both read the same value then write |
| Inconsistent analysis | Aggregate sees half-updated data | Multi-row read during concurrent writes |

**Interview Follow-up:**
- Which isolation level prevents phantom reads? (Serializable)
- How do you prevent lost updates? (Optimistic locking with a version column, or `SELECT ... FOR UPDATE`)

**Short Interview Answer:**
The main concurrency problems are dirty read, non-repeatable read, phantom read and lost update, and each one is what a higher isolation level is designed to prevent.

---

## 72. What is a deadlock?

**Answer:**
A deadlock is when two or more transactions each hold a resource the other needs, so neither can proceed and both wait forever. The DBMS detects this and picks one transaction as the victim to roll back. It can happen with row locks, table locks, or even index page locks.

**Example:**
T1 locks account A then wants B; T2 locks account B then wants A → both blocked forever.

**Interview Follow-up:**
- Does Spring handle this? (Usually yes — it retries on deadlock/lock timeout if configured)
- Are deadlocks rare in practice? (Not that rare, but cheap to recover from since only one transaction rolls back)

**Short Interview Answer:**
A deadlock is when transactions hold each other's needed resources in a circular wait, so none can finish and the DBMS must abort one of them.

---

## 73. What are the conditions for deadlock?

**Answer:**
Four conditions must all hold simultaneously: mutual exclusion, hold and wait, no preemption, and circular wait. If the DBMS prevents even one of them, deadlock becomes impossible. That's the basis of every prevention technique.

**Example:**
Waiting-die and wound-wait schemes break the no-preemption condition by aborting the younger transaction when there's a conflict.

**Interview Follow-up:**
- Name them. (Mutual exclusion, hold and wait, no preemption, circular wait)
- Which is easiest to break? (Circular wait — enforce a global lock ordering)

**Short Interview Answer:**
Deadlock requires mutual exclusion, hold-and-wait, no preemption and circular wait — breaking any one of them prevents deadlock.

---

## 74. How can deadlocks be prevented?

**Answer:**
Prevention means breaking one of the four conditions before they happen. The most practical techniques are lock ordering (impose a global order on all locks to kill circular wait), and timestamp schemes like wait-die and wound-wait which abort one transaction on conflict. Timeouts are a simple fallback.

**Example:**
Always lock the account with the lower ID first — then T1 and T2 can never hold-and-wait in opposite directions.

**Interview Follow-up:**
- Prevention vs detection? (Prevention avoids the deadlock; detection allows it and then recovers)
- Why prefer prevention in high-throughput systems? (A rolled-back transaction wastes all its work)

**Short Interview Answer:**
Deadlock is prevented by breaking one of the four conditions — most practically by acquiring locks in a fixed global order, or using wait-die / wound-wait timestamps.

---

## 75. How can deadlocks be detected and recovered?

**Answer:**
Detection lets the deadlock happen and then identifies it. The DBMS maintains a wait-for graph (edge T1 → T2 when T1 waits for T2's lock) and runs cycle detection; if a cycle exists, it picks a victim. Recovery means rolling back the victim transaction — usually the one with the least work or the lowest cost — and letting it retry.

**Example:**
If T1 waits for T2 and T2 waits for T1, the wait-for graph has a cycle, so the DBMS aborts T2 and restarts it later.

**Interview Follow-up:**
- What makes a good victim? (Cheap to roll back — fewest changes; avoid starvation by choosing the youngest)
- Which DBMS use this? (MySQL InnoDB uses wait-for graph with victim selection)

**Short Interview Answer:**
Detection builds a wait-for graph and looks for a cycle; recovery rolls back the chosen victim transaction and lets the application retry.

---

## 76. What are isolation levels in DBMS?

**Answer:**
Isolation levels define how much transaction interference is allowed. From lowest to highest: Read Uncommitted, Read Committed, Repeatable Read, and Serializable. Standard SQL defines these three anomaly types they progressively prevent; PostgreSQL also adds Repeatable Read with snapshot behaviour.

**Example:**

| Level | Dirty read | Non-repeatable read | Phantom | Speed |
|---|---|---|---|---|
| Read Uncommitted | Allowed | Allowed | Allowed | Fastest |
| Read Committed | Prevented | Allowed | Allowed | Fast |
| Repeatable Read | Prevented | Prevented | Allowed* | Medium |
| Serializable | Prevented | Prevented | Prevented | Slowest |

\*PostgreSQL's Repeatable Read actually prevents phantoms.

**Interview Follow-up:**
- What does MySQL default to? (Repeatable Read, which also prevents phantoms via next-key locks)
- Which level do most apps use? (Read Committed — like Oracle/Postgres defaults)

**Short Interview Answer:**
Isolation levels are Read Uncommitted, Read Committed, Repeatable Read and Serializable, and each higher level removes more anomalies at the cost of concurrency.

---

## 77. What is cascading rollback?

**Answer:**
Cascading rollback is when a transaction fails and the DBMS must undo its changes, but other transactions have already read that uncommitted data. Those transactions also have to be rolled back, and this can chain further. That's why long-running transactions are dangerous — a single failure can undo a lot of work.

**Example:**
T1 updates A and T2 reads A, then T1 crashes. Because T2's result depends on uncommitted data, T2 must also roll back.

**Interview Follow-up:**
- How to avoid it? (Keep transactions short, use higher isolation levels or MVCC)
- Does MVCC avoid cascading rollback? (Yes — readers see old committed versions, so no cascading undo needed)

**Short Interview Answer:**
Cascading rollback is when one transaction's failure forces the rollback of other transactions that had read its uncommitted data.

---

## 78. What is a checkpoint?

**Answer:**
A checkpoint is a point where the DBMS flushes all dirty buffers from memory to disk and records a special log entry. It's used to bound recovery time — after a crash, the system only needs to replay the log from the last checkpoint instead of the beginning. Checkpoints also help release transaction locks safely.

**Example:**
If the last checkpoint was at 10:00 and the DB crashed at 10:15, recovery replays only the log from 10:00 onwards.

**Interview Follow-up:**
- Fuzzy checkpoint? (Incremental — writes only some dirty pages, common in real DBMSs)
- How often are checkpoints triggered? (Based on log size or time, DBMS-managed)

**Short Interview Answer:**
A checkpoint is a marker where the DBMS flushes dirty pages to disk so recovery only needs to replay the log after the last checkpoint.

---

## 79. What is write-ahead logging (WAL)?

**Answer:**
WAL is the rule that a log record must be written to stable (durable) storage before the actual data page is written to disk. It means the log is always ahead of the data, so after any crash the DBMS can redo completed work and undo incomplete work. WAL is what makes durability and atomicity possible.

**Example:**
Before writing the updated page for a commit, the DBMS first makes sure the corresponding log record is safely on disk.

**Interview Follow-up:**
- What does "stable storage" mean? (Storage that survives power loss — not just OS cache)
- Which level of ACD does WAL guarantee? (Atomicity and Durability)

**Short Interview Answer:**
WAL means logging a change durably before applying it to the data pages, which lets recovery redo committed work and undo incomplete work after a crash.

---

## 80. What is ARIES recovery algorithm?

**Answer:**
ARIES is the redo-undo recovery algorithm used by most modern DBMSs. It works in three passes: analysis to reconstruct what was in flight at crash time, redo to reapply all changes (even lost ones), and undo to roll back uncommitted transactions using compensation log records. Because it redoes everything first, even committed data that hadn't reached disk is restored.

**Example:**
If a committed transaction's page write was still in the OS buffer when the crash happened, ARIES redoes it from the log — so the commit isn't lost.

**Interview Follow-up:**
- Why analysis pass first? (To find the dirty page table and active transactions)
- What is a CLR? (Compensation log record — logs the undo of an undo so recovery doesn't undo twice)

**Short Interview Answer:**
ARIES recovers in three phases — analysis, redo and undo — using the WAL, and it uses CLRs so undo operations are idempotent.

## 81. What is file organization in DBMS?

**Answer:**
File organization is how records and tables are physically arranged and accessed on disk. It decides how fast rows are found and how writes are handled. The DBMS supports multiple organizations, and you can even combine them in one database — for example a heap table for writes plus a B+ tree index for reads.

**Example:**
InnoDB stores table data in a clustered index (a B+ tree keyed by primary key), so "the file" and "the index" are effectively the same thing.

**Interview Follow-up:**
- Why does disk layout matter so much? (Disk seeks are slow; sequential access is far faster than random)
- What does the buffer pool do? (Caches hot pages in memory to avoid disk access)

**Short Interview Answer:**
File organization is the physical arrangement of records on disk — heap, sequential, hashed or indexed — and it directly affects read and write performance.

---

## 82. What are the types of file organization?

**Answer:**
The main types are sequential (unordered file, records appended in order), indexed (hashed or B+ tree based on a search key), and heap. Indexed sequential files are the classic option; modern systems use heaps for OLTP and B+ tree indexes for reads.

**Example:**

| Method | Access | Use case |
|---|---|---|
| Sequential / Unordered | Only full scan | Batch logs, archive |
| Indexed (B+ tree) | Fast equality + range | Primary lookups, OLTP |
| Indexed (Hash) | Fast equality only | Exact-match lookups |
| Heap | Full scan + fast insert | Write-heavy staging tables |

**Interview Follow-up:**
- Why can't hash indexes do range queries? (Hashing destroys order)
- Which is used for clustered storage? (B+ tree, because it supports range scans)

**Short Interview Answer:**
File organizations are sequential, indexed-sequential (B+ tree or hash), and heap — chosen based on whether you need fast lookups, range scans, or fast inserts.

---

## 83. What is indexing?

**Answer:**
Indexing is the technique of building an auxiliary data structure to speed up row retrieval. Instead of scanning every page, the DBMS navigates the index to the exact location. B+ tree is the default because it gives both O(log n) equality lookups and ordered range scans.

**Example:**
`CREATE INDEX idx_email ON users(email);` — without it, `WHERE email = ?` scans the table; with it, it's a tree lookup.

**Interview Follow-up:**
- Clustered vs non-clustered? (Clustered determines row storage order; only one per table)
- When should you not index? (Low-cardinality columns, very small tables, frequently updated columns)

**Short Interview Answer:**
Indexing builds an auxiliary structure — usually a B+ tree — so the DBMS can find rows in logarithmic time instead of scanning the whole table.

---

## 84. What are the types of indexes?

**Answer:**
Common types are clustered/non-clustered, primary/secondary, single-column/composite, full-text, bitmap and hash indexes. The primary distinction is storage order (clustered) versus just a lookup structure (non-clustered). Choose based on query patterns, not on the number of columns.

**Example:**

| Index | Use case |
|---|---|
| Clustered | Table storage order — InnoDB default on PK |
| Non-clustered | Secondary lookup, stores row pointer |
| Composite | Multi-column WHERE/JOIN/ORDER BY |
| Full-text | `MATCH ... AGAINST` text search |
| Bitmap | Low-cardinality columns like gender, status |
| Hash | Exact equality only |

**Interview Follow-up:**
- Column order in composite index? (Leftmost prefix rule — `(a,b)` helps queries on `a`, not on `b` alone)
- Covering index? (Index that contains all columns needed, so the table isn't touched)

**Short Interview Answer:**
Indexes can be clustered or non-clustered, single or composite, plus full-text, bitmap and hash types — pick them based on your query patterns.

---

## 85. What is a dense index?

**Answer:**
A dense index has an entry for every record in the file, and the index key points directly to the record's disk address. Lookup is fast because no binary search inside a block is needed, but the index itself can become large. It's suitable for small, static files.

**Example:**
For a 1000-row file, a dense index has 1000 entries, each mapping directly to one row.

**Interview Follow-up:**
- Advantage? (Direct record access — no need to scan the block)
- When is it used? (Small tables where every record is frequently accessed)

**Short Interview Answer:**
A dense index contains an entry for every record, pointing directly to its address, so lookups are fast but the index is larger.

---

## 86. What is a sparse index?

**Answer:**
A sparse index has entries only for some records — typically the first record of each block. It's much smaller, but the DBMS may need to read the whole block to find the exact row. It's good for large files that are scanned sequentially most of the time.

**Example:**
If a file has 100 blocks of 10 records, the sparse index has just 100 entries, and each points to a block start.

**Interview Follow-up:**
- Dense vs sparse? (Dense = every record; sparse = every block)
- Which does B+ tree use? (Sparse at the non-leaf levels — only keys and pointers to child nodes)

**Short Interview Answer:**
A sparse index stores an entry only per block, so it's compact but requires scanning within the block to find the exact record.

---

## 87. What is a B-tree?

**Answer:**
A B-tree is a balanced search tree where keys and record data are stored together in every node, keeping the tree height small. Because all internal nodes are full, a B-tree holds a lot of keys near the root, so very few disk reads are needed. Disks read in blocks, which is why B-trees work so well for databases.

**Example:**
A B-tree of order 5 stores 4 keys per node — for a million rows the tree height is only about 4, so a lookup costs ~4 disk reads.

**Interview Follow-up:**
- What is the height formula? (log_m(N))
- B-tree vs B+ tree? (B-tree stores data everywhere; B+ tree stores data only in leaves)

**Short Interview Answer:**
A B-tree is a balanced tree that stores keys and data together in every node, giving O(log n) lookups with very few disk reads.

---

## 88. What is a B+ tree?

**Answer:**
A B+ tree is the variant actually used by most databases. Internal nodes store only keys and child pointers; all data lives in the leaf nodes, and leaves are linked together in a sorted order. That leaf-level linked list makes range queries and full scans fast, and it also means internal nodes are smaller so more keys fit per disk block.

**Example:**
`SELECT * FROM orders WHERE customer_id = 5 AND amount > 100` finds the starting leaf via the tree, then just walks the leaf chain.

**Interview Follow-up:**
- Why do leaves need to be linked? (Efficient range and ordered scans)
- Clustered index? (The B+ tree leaf holds the whole row, so there's only one such tree per table)

**Short Interview Answer:**
A B+ tree keeps all data in linked leaf nodes and only keys in internal nodes, which makes it the standard choice for indexes because it supports fast equality and range lookups.

---

## 89. What is hashing in DBMS?

**Answer:**
Hashing is a file organization where a hash function converts the search key into a bucket address, so you go straight to the right bucket instead of searching. It gives O(1) average access for equality lookups but cannot support range queries or ordered scans. It's useful when you only ever search by an exact key.

**Example:**
`hash(101) % 8 = 5` → record with id 101 is stored in bucket 5.

**Interview Follow-up:**
- Overflow? (When a bucket fills, you need chaining or open addressing)
- Hash vs B+ tree? (Hash = equality only; B+ tree = equality plus range)

**Short Interview Answer:**
Hashing uses a hash function to map a key directly to a bucket, giving O(1) equality lookups but no support for range queries.

---

## 90. What is extendible hashing?

**Answer:**
Extendible hashing is a dynamic hashing scheme that grows gracefully by adding a new bucket when one overflows. It maintains a global directory of pointers to buckets, and only a single bucket is split instead of re-hashing everything. So it handles a growing dataset with minimal reorganization.

**Example:**
The directory doubles from 4 to 8 pointers, and only the overflowing bucket is split into two — the other buckets stay where they are.

**Interview Follow-up:**
- What is the split algorithm? (Use the least significant bits of the hash value as bucket indices)
- Advantage over static hashing? (No need to rehash the entire file when it grows)

**Short Interview Answer:**
Extendible hashing handles growth by splitting only the overflowing bucket and doubling a global directory, so existing data isn't rehashed.

---

## 91. What is linear hashing?

**Answer:**
Linear hashing also grows dynamically, but instead of a directory it uses a single global pointer. When a bucket overflows, the pointer increments and a new bucket is added at the end; when the pointer catches up to the highest bucket, everything doubles. It uses less memory than extendible hashing but requires scanning to decide which bucket to split.

**Example:**
With 4 buckets and the pointer at 2, bucket 3 splits into 3 and 7; when the pointer reaches 8, the file doubles to 8 buckets.

**Interview Follow-up:**
- Linear vs extendible? (Linear uses one pointer and rehashes linearly; extendible uses a directory with O(1) lookup)
- Both used in practice? (Both are academic/legacy — B+ trees dominate real systems)

**Short Interview Answer:**
Linear hashing grows by splitting one bucket at a time using a single global level pointer, and the file doubles when the pointer wraps around.

---

## 92. What is query processing?

**Answer:**
Query processing is the full lifecycle the DBMS goes through for a SQL statement: parsing, translation, optimization and execution. Parsing checks syntax and builds a tree; translation converts names to internal IDs; optimization picks the cheapest plan; execution runs the operators and returns rows. Knowing this helps you read an `EXPLAIN` plan and understand where time goes.

**Example:**
For `SELECT * FROM orders WHERE status='PAID'`, the steps are parse → resolve `status` via the catalog → choose index scan vs full scan → fetch rows.

**Interview Follow-up:**
- Compilation stages? (Parse, translate, optimize, execute)
- What does the plan cache do? (Reuses the optimized plan for repeated queries)

**Short Interview Answer:**
Query processing is parse → translate → optimize → execute, and understanding each stage explains why the database picks a particular plan for a query.

---

## 93. What is query optimization?

**Answer:**
Query optimization chooses the best way to execute a query among many equivalent options. The DBMS considers join order, access paths, and join algorithms, then estimates which plan costs the least. Optimizers are rule-based plus cost-based, and for real data the cost-based part does most of the work.

**Example:**
For three tables joined together, there are many join orders; the optimizer picks the one that avoids scanning the largest table first.

**Interview Follow-up:**
- Why is it important? (A bad plan can be 100x slower on the same SQL)
- Heuristic vs cost-based? (Heuristic uses fixed rules; cost-based uses statistics like rows and selectivity)

**Short Interview Answer:**
Query optimization picks the cheapest execution plan for a SQL statement by evaluating join orders, indexes and join methods using table statistics.

---

## 94. What is a query execution plan?

**Answer:**
An execution plan is the step-by-step recipe the optimizer chose to run a query — which index to use, in what order to join tables, and what algorithms to apply. You can see it with `EXPLAIN` or `EXPLAIN ANALYZE` in MySQL/PostgreSQL. Reading this plan is one of the most practical skills for a backend developer.

**Example:**
```sql
EXPLAIN SELECT c.name, o.amount
FROM orders o JOIN customers c ON c.id = o.customer_id
WHERE o.status = 'PAID';
-- Nested Loop  (rows=1200)  ->  Index Scan idx_orders_status  ->  Index Scan customers_pkey
```

**Interview Follow-up:**
- EXPLAIN vs EXPLAIN ANALYZE? (EXPLAIN estimates; ANALYZE actually executes and shows real rows/timing)
- What is a Nested Loop? (For each row of the outer table, probe the inner table — good for small outer sets with indexes)

**Short Interview Answer:**
An execution plan is the step-by-step method the DBMS uses to run your query — you inspect it with EXPLAIN to find full scans, bad join orders or missing indexes.

---

## 95. What is cost-based optimization?

**Answer:**
Cost-based optimization compares alternative plans using estimated costs like I/O, CPU and memory, driven by table statistics (rows, distinct values, histograms). If statistics are stale or missing, the optimizer picks a bad plan — which is why `ANALYZE` or updating statistics matters. It complements rule-based heuristics.

**Example:**
The optimizer estimates `status = 'PAID'` matches 5% of rows; if it wrongly thinks it's 50%, it wrongly chooses a full table scan.

**Interview Follow-up:**
- How are costs estimated? (Rows × page I/O cost + CPU cost, using histograms and ndv)
- Does more data mean more accurate stats? (Yes — and stale stats are a common cause of slow queries)

**Short Interview Answer:**
Cost-based optimization picks the plan with the lowest estimated I/O and CPU cost, relying on table statistics — so stale statistics are a common cause of slow queries.

---

## 96. What is selectivity in DBMS?

**Answer:**
Selectivity is the fraction of rows in a table that match a given condition, between 0 and 1. The optimizer uses selectivity to estimate cardinality, which drives join order and access path decisions. High selectivity means few matching rows.

**Example:**
`WHERE status = 'PAID'` on an index with 5% of rows has selectivity 0.05, i.e. 5 out of 100 rows match.

**Interview Follow-up:**
- Low selectivity? (Most rows match — the optimizer may prefer a full scan over an index)
- Selectivity vs cardinality? (Selectivity = fraction; cardinality = actual number of rows)

**Short Interview Answer:**
Selectivity is the proportion of rows matching a condition, and the optimizer uses it with statistics to estimate how many rows a query will process.

---

## 97. What is cardinality in DBMS?

**Answer:**
Cardinality is the estimated number of rows a relation or expression will produce. The optimizer starts from base table cardinality and multiplies by selectivities to estimate each step's row count. It's the single most important input for choosing join order.

**Example:**
`orders` has 1,000,000 rows, `status='PAID'` selectivity 0.05 → estimated 50,000 rows → a hash join is now a better choice.

**Interview Follow-up:**
- Distinct from the SQL `CARDINALITY()` function? (Yes — SQL's returns an estimate of distinct values, e.g. used in DB link profiles)
- High vs low cardinality? (High = many distinct values, good for indexing; low = few, like boolean)

**Short Interview Answer:**
Cardinality is the estimated number of rows a query step will return, and the optimizer uses it together with selectivity to decide join order and access paths.

---

## 98. What is OLTP?

**Answer:**
OLTP (Online Transaction Processing) handles many small, frequent read/write transactions on day-to-day operations. It's designed for low latency and high concurrency on a small, current dataset — for example, order placement, payments and login. Its schema is usually normalized and indexed.

**Example:**
An e-commerce checkout: a few inserts/updates per user, thousands of concurrent users, sub-second response expected.

**Interview Follow-up:**
- Typical characteristics? (Row-oriented, primary key lookups, ACID transactions, 3NF)
- Why normalize for OLTP? (Less redundancy, faster writes on single rows)

**Short Interview Answer:**
OLTP is the workload of small, frequent, concurrent read/write transactions like payments and orders — it's low-latency and row-oriented.

---

## 99. What is OLAP?

**Answer:**
OLAP (Online Analytical Processing) handles complex, read-heavy analysis over aggregated historical data. Queries scan many rows, use joins and aggregations across dimensions, and can take seconds to minutes. Its schema is typically denormalized into star or snowflake schemas.

**Example:**
A dashboard showing revenue by region, quarter and product category for the last 3 years.

**Interview Follow-up:**
- Star vs snowflake? (Star denormalizes dimensions for speed; snowflake normalizes them to reduce redundancy)
- Cube vs relational OLAP? (MOLAP uses precomputed cubes; ROLAP uses relational tables)

**Short Interview Answer:**
OLAP is analytical processing over large historical datasets — read-heavy aggregations, often seconds long, using denormalized star/snowflake schemas.

---

## 100. What is the difference between DBMS and RDBMS?

**Answer:**
An RDBMS stores data as tables with rows, columns, keys and relationships, and supports SQL. A general DBMS could store anything — files, graphs, key-value pairs — so it may not have tables or SQL at all. Every relational DBMS is a DBMS, but not every DBMS is relational.

**Example:**

| Feature | DBMS (general) | RDBMS |
|---|---|---|
| Data structure | Any — files, graphs, KV | Tables with rows/columns |
| Access language | Proprietary / API | SQL (standard) |
| Relationships | Not enforced | Foreign keys enforce |
| Redundancy control | Manual | Normalization |
| Examples | MongoDB, Redis, Neo4j, file DB | MySQL, PostgreSQL, Oracle |

**Interview Follow-up:**
- Is MongoDB an RDBMS? (No, it's a document NoSQL DBMS)
- SQL vs NoSQL — when to pick which? (RDBMS for structured relational data and ACID; NoSQL for flexible schema, high write throughput, huge scale)

**Short Interview Answer:**
An RDBMS is a DBMS that stores data in related tables and uses SQL with keys and constraints; a general DBMS is broader and may store files, documents or graphs instead.

---

✅ **DBMS complete — 100/100 answered.**