<div align="center">
  <img src="https://cdn.worldvectorlogo.com/logos/prisma-2.svg" alt="Prisma Logo" width="200"/>
  <h1>Prisma ORM: The Ultimate A-Z Guide</h1>
  <p>
    <b>Master Database Queries with Prisma</b><br>
    <i>From schema design to complex transactions, explained step-by-step.</i>
  </p>
</div>

---

## 📚 Table of Contents
- [A. Prisma Model Basics](#a-prisma-model-basics-basics-a-z)
- [B. Basic Queries (CRUD)](#b-basic-query-a--z)
- [C. Relational Queries](#c-relational-query)
- [D. Filtering & Sorting](#d-filtering-where-query-a-z)
- [E. Pagination](#e-pagination)
- [F. Aggregation & Grouping](#h-aggregation-detailed)
- [G. Advanced Concepts (Nested & Transactions)](#i-advanced-transactions--nested-writes)
- [H. Raw SQL](#j-raw-sql-when-you-need-full-control)
- [I. Expert Queries (Upsert, Distinct, Soft Delete)](#k-expert-queries-master-level)

---

## 🅐 Prisma Model Basics (Basics A–Z)

Before querying, we need to define our database structure. In Prisma, this is done in the `schema.prisma` file.

### Example: Blog Application Schema

```prisma
model User {
  id        String   @id @default(uuid())
  name      String
  email     String   @unique
  posts     Post[]
  createdAt DateTime @default(now())
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())
}
```

### 🔍 Key Concepts Explained

| Concept | Description |
| :--- | :--- |
| **`model`** | Represents a table in your database. Here, `User` and `Post` are tables. |
| **`@id`** | Marks a field as the **Primary Key** (unique identifier for the record). |
| **`@default(uuid())`** | Automatically generates a unique string ID (UUID) when a record is created. |
| **`@unique`** | Ensures no two records can have the same value for this field (e.g., distinct emails). |
| **`?`** | Marks a field as **Optional** (nullable). `String?` means content can be text or `null`. |
| **`@default(now())`** | Sets the default value to the current timestamp when the record is created. |
| **`@relation`** | Defines a relationship (Foreign Key) between two models. Links `Post` to `User`. |

---

## 🅑 BASIC Query (A → Z)

Prisma queries fall into three main categories: **Read (find)**, **Write (create/update/delete)**, and **Aggregate**.

### 1️⃣ Create (Writing Data)
**Scenario:** A new user signs up.

```javascript
const user = await prisma.user.create({
  data: {
    name: "Mehefuj",
    email: "mehefuj@gmail.com",
  },
});
```
#### 🛠 Breakdown
* **`create`**: The method used to insert a new record into the database.
* **`data`**: An object containing the actual values you want to save. This is mandatory in `create` queries.

### 2️⃣ findUnique (Reading Specific Data)
**Scenario:** Logging in a user by their email.

```javascript
const user = await prisma.user.findUnique({
  where: { email: "mehefuj@gmail.com" },
});
```
#### 🛠 Breakdown
* **`findUnique`**: Retrieves exactly **one** record. It requires a unique field (like `@id` or `@unique`) to function.
* **`where`**: Specifies the condition to filter the data. Think of it as "Look for the user **where** the email equals X".

### 3️⃣ findFirst (Finding the First Match)
**Scenario:** Showing the very first published post on a dashboard.

```javascript
const post = await prisma.post.findFirst({
  where: { published: true },
});
```
#### 🛠 Breakdown
* **`findFirst`**: Returns the first record that matches the criteria. Unlike `findUnique`, this can be used on non-unique fields (like `published`).

### 4️⃣ findMany (Reading Multiple Records)
**Scenario:** Fetching a list of all published posts, newest first.

```javascript
const posts = await prisma.post.findMany({
  where: { published: true },
  orderBy: { createdAt: "desc" },
});
```
#### 🛠 Breakdown
* **`findMany`**: Returns an array of records matching the condition.
* **`orderBy`**: Sorts the results. `desc` means Descending (Newest to Oldest), `asc` means Ascending.

### 5️⃣ Update (Modifying Data)
**Scenario:** A user updates their profile name.

```javascript
const user = await prisma.user.update({
  where: { id: userId },
  data: { name: "Updated Name" },
});
```
* **`update`**: Modifies an existing record. Requires a `where` clause to find the specific item and `data` to define the changes.

### 6️⃣ Delete (Removing Data)
**Scenario:** A user deletes their account.

```javascript
await prisma.user.delete({
  where: { id: userId },
});
```
* **`delete`**: Permanently removes a record identified by the `where` clause.

---

## 🅒 Relational Query (Joins Made Easy)

### 1️⃣ Include (Fetching Related Data)
**Scenario:** Get a user and load all their posts automatically.

```javascript
const user = await prisma.user.findUnique({
  where: { id: userId },
  include: {
    posts: true,
  },
});
```
#### 🛠 Breakdown
* **`include`**: Tells Prisma to fetch related data (like a JOIN in SQL). Here, it grabs the `User` **AND** their `Post`s in a single query.

### 2️⃣ Select (Fetching Specific Fields)
**Scenario:** Get only User IDs and Names (optimize performance).

```javascript
const users = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
  },
});
```
#### 🛠 Breakdown
* **`select`**: Specifies exactly which columns to return. Helpful for reducing payload size.
* **Excluding Fields:** You cannot use `include` and `select` on the same level. To exclude a password, you select everything else or use a custom extension.

---

## 🅓 Filtering (WHERE Query A–Z)

### 1️⃣ String Filters (equals, contains, startsWith)
```javascript
const posts = await prisma.post.findMany({
  where: {
    title: {
      contains: "javascript",
      mode: "insensitive" // Ignores Case (e.g., 'JS' == 'js')
    }
  }
});
```
#### 🛠 Breakdown
* **`contains`**: Matches records where the field contains the substring.
* **`mode: "insensitive"`**: Ignores capitalization during the search.

### 2️⃣ List Filters (in, notIn)
```javascript
const users = await prisma.user.findMany({
  where: {
    id: { in: ["1", "2", "3"] }
  }
});
```
* **`in`**: Checks if the value exists within the provided array.

### 3️⃣ Logical Operators (OR / AND)
```javascript
const posts = await prisma.post.findMany({
  where: {
    OR: [
      { published: true },
      { title: { contains: "Next.js" } }
    ]
  }
});
```
* **`OR`**: Returns records that match **at least one** of the conditions in the list.

---

## 🅔 Pagination

**Scenario:** Loading data in pages (infinite scroll or page numbers).

```javascript
const data = await prisma.post.findMany({
  skip: 20,
  take: 10,
});
```
#### 🛠 Breakdown
* **`skip`**: Skips the first N records (e.g., skip first 20).
* **`take`**: Limits the number of records returned (e.g., get next 10).

---

## 🅗 AGGREGATION (Detailed)

### 1️⃣ aggregate() (Min, Max, Avg, Count)
```javascript
const stats = await prisma.post.aggregate({
  _count: true,
  _avg: { id: true }, // N/A for UUIDs, useful for Integers
  _min: { createdAt: true },
  _max: { createdAt: true },
});
```

### 2️⃣ groupBy() (Reports)
**Scenario:** How many posts does each user have?

```javascript
const report = await prisma.post.groupBy({
  by: ["authorId"],
  _count: { id: true },
  orderBy: {
    authorId: "asc"
  }
});
```
#### 🛠 Breakdown
* **`groupBy`**: Groups records by a specific field (e.g., `authorId`).
* **`by`**: The field(s) used to group the data. Mandatory for `groupBy`.

---

## 🅘 Advanced Transactions & Nested Writes

### 1️⃣ Nested Create
**Scenario:** Create a User and their first two Posts instantly.

```javascript
await prisma.user.create({
  data: {
    name: "Mehefuj",
    email: "mehefuj@gmail.com",
    posts: {
      create: [
        { title: "First Post" },
        { title: "Second Post" },
      ]
    }
  }
});
```
* **Nested Write:** You can create related records inside the parent's `data` object directly.

### 2️⃣ Transactions
**Scenario:** Transfer money (A → B). If one fails, both fail.

```javascript
await prisma.$transaction(async (tx) => {
  await tx.user.update({
    where: { id: senderId },
    data: { balance: { decrement: 500 } }
  });

  await tx.user.update({
    where: { id: receiverId },
    data: { balance: { increment: 500 } }
  });
});
```
#### 🛠 Breakdown
* **`$transaction`**: Ensures all queries inside run successfully. If any error occurs, all changes are rolled back.
* **`decrement` / `increment`**: Atomic operations to safely increase or decrease numerical values.

---

## 🅙 Raw SQL (When you need full control)

### 1️⃣ Raw SELECT
```javascript
const data = await prisma.$queryRaw`
  SELECT * FROM "User" WHERE email = ${email};
`;
```

### 2️⃣ Raw INSERT/EXECUTE
```javascript
await prisma.$executeRaw`
  INSERT INTO "User" ("id", "email", "name")
  VALUES (${id}, ${email}, ${name});
`;
```

---

## 🅚 Expert Queries (Master Level)

Here are the specific, powerful queries for advanced scenarios.

### 1️⃣ UPSERT (Update if Exists, else Create)
**Scenario:** Updating a user profile if they exist, or creating a new one if they don't.

```javascript
await prisma.user.upsert({
  where: { email: "mehefuj@gmail.com" },
  create: { email: "mehefuj@gmail.com", name: "Mehefuj" },
  update: { name: "Updated Name" }
});
```
* **`upsert`**: A combination of Update + Insert.

### 2️⃣ Soft Delete (Don't actually delete)
**Scenario:** Hiding data instead of removing it from the DB.

```javascript
await prisma.user.update({
  where: { id },
  data: { deletedAt: new Date() }
});
```
* **Concept:** You just update a `deletedAt` timestamp instead of using `delete()`.

### 3️⃣ Full-Text Search (PostgreSQL)
**Scenario:** Searching for "nextjs" and "prisma" in titles.

```javascript
await prisma.post.findMany({
  where: {
    title: { search: "nextjs & prisma" }
  }
});
```

### 4️⃣ DISTINCT Query
**Scenario:** Get a list of all unique emails.

```javascript
await prisma.user.findMany({
  distinct: ["email"],
});
```

### 5️⃣ EXISTS Query (Relation Check)
**Scenario:** Find all users who have at least one published post.

```javascript
await prisma.user.findMany({
  where: {
    posts: { some: { published: true } }
  }
});
```

### 6️⃣ Complex Filtering (AND + OR)
**Scenario:** Find published posts that mention "JS" OR "Tutorial".

```javascript
await prisma.post.findMany({
  where: {
    AND: [
      { published: true },
      {
        OR: [
          { title: { contains: "js" } },
          { content: { contains: "tutorial" } }
        ]
      }
    ]
  }
});
```

---

<div align="center">
  <sub>Made with ❤️ for the Developer Community</sub>
</div>
