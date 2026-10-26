# S19–S20: Android Data Persistence, SQLite, Content Providers & Firebase
### Unit III — Kotlin + XML Views | Concepts, Practical Examples & Revision

This guide covers the Unit-III topics listed for S19, S20 and Unit-III revision: local data persistence, SQLite CRUD, Content Providers and Cursors, Firebase Realtime Database, offline/online storage decisions, and security.

> **Learning approach:** Understand the problem each technology solves, then practise its basic flow in Android Studio. Adapt imports, package names, dependencies and error handling to your project.

## Learning Outcomes

- Explain data persistence and why apps need to retain data.
- Compare Android storage mechanisms and choose an appropriate option.
- Explain database tables, rows, columns and basic database design.
- Describe `SQLiteOpenHelper` and database creation.
- Explain CRUD operations.
- Describe Content Providers, `ContentResolver`, URIs and Cursors.
- Read records using cursor navigation methods.
- Explain Firebase Realtime Database and cloud synchronization.
- Compare local and cloud storage, including offline/online behaviour and security.
- Revise Unit-III concepts together.

## Session Map

| Session / block | Main focus |
|---|---|
| **S19 — Part A** | Data persistence, Android storage options, local storage, SharedPreferences and SQLite basics |
| **S19 — Part B** | Database design, SQLiteOpenHelper, database creation, CRUD and optimization basics |
| **S20 — Part A** | Data sharing, Content Providers, ContentResolver, URIs, structured data and Cursors |
| **S20 — Part B** | Firebase, cloud storage, real-time synchronization, offline/online choices and security |
| **Unit-III Revision** | Layouts, widgets, RecyclerView, persistence, SQLite, Content Providers, Firebase and interview questions |

---

# Part 1 — Data Persistence & Android Storage

## 1. What Is Data Persistence?

**Data persistence** means keeping information available beyond the immediate lifetime of a screen or app process. For example, a notes app should not lose saved notes whenever the user closes a screen.

Values held only in temporary variables can disappear when the app process ends.

Examples:
- Remembering app settings.
- Saving notes or student records.
- Keeping a shopping cart between app launches.
- Synchronizing user data across devices.

## 2. Android Storage Options

| Mechanism | Common use | Main consideration |
|---|---|---|
| In-memory variables | Temporary values while code runs | Not durable storage |
| SharedPreferences | Small key-value settings | Not designed for large structured records |
| Internal app storage | App-private files | Normally private to the app |
| Local files | Documents, images and file content | The app manages file structure and access |
| SQLite | Structured records and queries | Uses tables and SQL |
| Content Provider | Sharing/accessing structured data through a standard interface | Provider rules and permissions apply |
| Firebase Realtime Database | Cloud data and real-time synchronization | Requires secure configuration and network-aware design |

“Local storage” is a broad term. Internal app files and SQLite databases are both local storage approaches, but they solve different problems.

## 3. SharedPreferences

`SharedPreferences` stores small pieces of data as key-value pairs, such as a theme preference or setting.

```kotlin
val preferences = getSharedPreferences("app_settings", MODE_PRIVATE)

// Save a setting
preferences.edit()
    .putBoolean("notifications_enabled", true)
    .apply()

// Read a setting
val enabled = preferences.getBoolean("notifications_enabled", false)
```

Use it for small preferences, not large collections of relational records. Do not store passwords or sensitive secrets as plain preference values.

## 4. Internal and Local Storage

- **Internal app files:** suitable for app-private files and content.
- **SQLite database:** suitable for structured data that benefits from tables and queries.
- **SharedPreferences:** suitable for small settings.
- **Cloud database:** suitable when data needs to be available across devices or synchronized through a service.

Ask these questions when choosing storage:
1. Is this a small setting or a collection of records?
2. Does the data need filtering, sorting or relationships?
3. Must another app access the data?
4. Must data synchronize across devices?
5. What privacy and access rules are required?

---

# Part 2 — SQLite Database & CRUD Architecture

## 5. SQLite Introduction

SQLite is a lightweight relational database engine used for local structured data. Information is organized into tables containing rows and columns.

| Term | Meaning | Example |
|---|---|---|
| Database | Organized collection of data | Student database |
| Table | Records of one kind | `students` |
| Row | One record | One student's details |
| Column | One field/attribute | `name`, `course` |
| Primary key | Field identifying a record | `id` |
| Query | Instruction to read or modify data | `SELECT`, `INSERT` |

Example table:

| id | name | course |
|---:|---|---|
| 1 | Asha | CSE |
| 2 | Rahul | IT |
| 3 | Meera | ECE |

## 6. Basic Database Design

Before creating a database:
1. Identify what information the app must store.
2. Group related information into tables.
3. Choose columns and suitable data types.
4. Identify a primary key for records.
5. Avoid unnecessary duplication.
6. Consider which records need to be searched or updated.

## 7. SQLiteOpenHelper

`SQLiteOpenHelper` manages database creation and version management for a SQLite database.

- `onCreate()` — called when the database is first created.
- `onUpgrade()` — used when the database version changes and a migration is required.

```kotlin
class StudentDbHelper(context: Context) :
    SQLiteOpenHelper(context, "students.db", null, 1) {

    override fun onCreate(db: SQLiteDatabase) {
        db.execSQL(
            "CREATE TABLE students (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                "name TEXT NOT NULL, " +
                "course TEXT NOT NULL)"
        )
    }

    override fun onUpgrade(
        db: SQLiteDatabase,
        oldVersion: Int,
        newVersion: Int
    ) {
        // Add the required migration when the schema changes.
    }
}
```

This is a teaching example. A real upgrade path should preserve existing user data and handle supported schema transitions. Avoid dropping tables in production upgrades unless data loss is explicitly intended.

## 8. Database Creation Flow

```text
App needs database
       |
       v
Create/use SQLiteOpenHelper
       |
       v
Request readable or writable database
       |
       v
Database file checked
       |
       +--> New database: onCreate()
       |
       +--> Existing database: use existing schema
       |
       +--> Version changed: onUpgrade() as needed
```

## 9. CRUD Operations

CRUD means **Create, Read, Update and Delete**.

| Operation | Purpose | Typical SQL |
|---|---|---|
| Create | Add a record | `INSERT` |
| Read | Retrieve records | `SELECT` |
| Update | Change a record | `UPDATE` |
| Delete | Remove a record | `DELETE` |

### Insert a student

```kotlin
val dbHelper = StudentDbHelper(this)
val db = dbHelper.writableDatabase

val values = ContentValues().apply {
    put("name", "Asha")
    put("course", "CSE")
}

val newRowId = db.insert("students", null, values)
```

### Read students

```kotlin
val db = dbHelper.readableDatabase
val cursor = db.query(
    "students",
    arrayOf("id", "name", "course"),
    null, null, null, null,
    "name ASC"
)

cursor.use {
    while (it.moveToNext()) {
        val id = it.getLong(it.getColumnIndexOrThrow("id"))
        val name = it.getString(it.getColumnIndexOrThrow("name"))
        val course = it.getString(it.getColumnIndexOrThrow("course"))
        // Use the values, for example, to build a result list.
    }
}
```

### Update a student

```kotlin
val values = ContentValues().apply {
    put("course", "Information Technology")
}

val rowsUpdated = db.update(
    "students",
    values,
    "id = ?",
    arrayOf("1")
)
```

### Delete a student

```kotlin
val rowsDeleted = db.delete(
    "students",
    "id = ?",
    arrayOf("1")
)
```

Use selection arguments (`?`) instead of inserting raw user input into SQL strings. This reduces SQL injection risk and handles values more safely.

## 10. SQLite Best Practices & Optimization Basics

- Query only the columns required.
- Use selection arguments instead of concatenating user input into SQL.
- Close cursors and other resources; Kotlin's `use {}` helps close a cursor.
- Keep database work off the main UI thread when it could take noticeable time.
- Use transactions for groups of operations that should succeed or fail together.
- Add indexes for frequently queried columns when justified by query patterns.
- Plan database version upgrades carefully.
- Avoid unnecessary repeated queries.
- Consider Room when a project uses the AndroidX Room persistence library; this session's core focus remains the listed SQLite concepts.

---

# Part 3 — Content Providers, URIs & Cursors

## 11. Why Share Data Between Apps?

An app may need to expose selected structured data to another app or access data exposed by another app. A Content Provider offers a standard interface for managing and sharing such data.

Examples can include contacts or media data when the provider and permissions allow access.

## 12. Content Provider Architecture

```text
App / Client
    |
    v
ContentResolver
    |
    v
Content URI
    |
    v
Content Provider
    |
    v
Underlying data source
```

| Component | Role |
|---|---|
| Content Provider | Exposes data operations through a defined interface |
| ContentResolver | Client-side API used to query or modify provider data |
| Content URI | Identifies the provider and the data being addressed |
| Cursor | Provides access to rows returned by many structured queries |

A provider controls what data is exposed and which permissions are required. A URI by itself does not grant permission.

## 13. URI Concepts

A content URI commonly follows this pattern:

```text
content://authority/path
```

- `content://` — scheme used for content URIs.
- `authority` — identifies the provider.
- `path` — identifies a collection or a specific item.

A URI may address a collection or a single record. Use the URI and permissions specified by the provider's documentation.

## 14. ContentResolver Concept

A client commonly uses `ContentResolver` to query, insert, update or delete data exposed by a provider.

```kotlin
val contentUri: Uri = /* URI supplied by the provider */

val cursor = contentResolver.query(
    contentUri,
    null,  // projection: columns to return
    null,  // selection filter
    null,  // selection arguments
    null   // sort order
)

cursor?.use {
    while (it.moveToNext()) {
        // Read requested columns from each result row.
    }
}
```

The actual URI, columns, permissions and supported operations depend on the provider. Do not assume every provider supports every operation.

## 15. Cursor Introduction

A `Cursor` provides access to rows returned by a database or content query. It has a current position and can move between records.

| Method | Purpose |
|---|---|
| `moveToFirst()` | Move to the first row; returns false if there is no row |
| `moveToNext()` | Move to the next row; returns false when no next row exists |
| `moveToPosition(position)` | Move to a specific zero-based position |
| `getCount()` | Return the number of rows in the result |
| `getColumnIndexOrThrow(name)` | Find a column index or throw if it is absent |

### Cursor best practices

- Check whether the query returned a cursor.
- Use `moveToNext()` in a loop.
- Use `getColumnIndexOrThrow()` or check column indices before reading.
- Close the cursor; Kotlin's `use {}` is convenient.
- Do not assume a query returned at least one row.
- Request only the columns needed.
- Respect provider permissions and handle access failures.

---

# Part 4 — Firebase & Cloud Data Management

## 16. Cloud Database Introduction

A cloud database stores data on infrastructure accessed over a network. It can help make data available to multiple devices or users, depending on the service and its security rules.

Cloud data management introduces network availability, access control, synchronization, conflicts and data consistency considerations.

## 17. Firebase Realtime Database

Firebase Realtime Database is a cloud-hosted database that stores data as a JSON-like tree and supports real-time synchronization between connected clients.

```text
Android App
    |
    v
Firebase SDK
    |
    v
Realtime Database service
    |
    +----> Authorized client A
    |
    +----> Authorized client B
```

The service can propagate changes to connected clients when configured and authorized to access the relevant data.

## 18. Basic Firebase Data Structure

A conceptual JSON-like structure:

```json
{
  "students": {
    "student_1": {
      "name": "Asha",
      "course": "CSE"
    },
    "student_2": {
      "name": "Rahul",
      "course": "IT"
    }
  }
}
```

This demonstrates data organization, not a complete Firebase setup. Implementation requires configuring a Firebase project, adding the Android app, including the appropriate SDK, and defining secure database rules.

## 19. Cloud Storage & Real-Time Synchronization

A simplified flow:

1. The app establishes the identity/context required by database rules.
2. The app reads or writes data through the Firebase SDK.
3. The service evaluates configured security rules.
4. Authorized changes are stored.
5. Connected clients listening to the relevant data can receive updates.
6. The UI updates based on the received data.

The exact behaviour depends on SDK configuration, rules, connectivity and the API used.

## 20. Offline vs Online Storage

| Aspect | Local storage | Cloud storage |
|---|---|---|
| Network required for basic local access | Usually no | Usually yes for server communication |
| Data availability | Primarily on that device/app installation | Can be available to authorized clients across devices |
| Synchronization | Must be designed if multiple copies exist | Can be supported by the cloud service |
| Main concerns | Device loss, local data lifecycle, migrations | Network failures, rules, authentication, conflicts |
| Examples | SharedPreferences, files, SQLite | Firebase Realtime Database |

Do not assume every cloud operation succeeds instantly or that every local database automatically synchronizes with the cloud.

## 21. Data Consistency Basics

When data exists in more than one place, the app must consider whether those copies agree.

Ask:
- What if two devices edit the same record?
- What should the user see when the device is offline?
- When should local changes be uploaded?
- How are failures retried?
- Which update should win if changes conflict?

A production app may require conflict-resolution rules, retry handling, timestamps or other mechanisms based on its requirements.

## 22. Security Considerations

- Configure database security rules; do not leave data publicly writable.
- Authenticate users when required by the app's access model.
- Grant only the access each user needs.
- Never place private service credentials in a client app.
- Validate data and protect sensitive fields.
- Test both allowed and denied access.
- Hiding a UI control does not secure the underlying data.

---

# Part 5 — Choosing Local or Cloud Storage

| Requirement | Possible starting point |
|---|---|
| A small app setting | SharedPreferences |
| App-private files | Internal app storage |
| Structured records queried with SQL | SQLite |
| Standard access to data shared through another app/provider | ContentResolver + Content Provider |
| Data synchronized to authorized users/devices | Firebase or another suitable cloud service |
| Both offline access and cloud synchronization | Local storage plus a deliberately designed synchronization strategy |

There is no single best storage option for every app. Consider data structure, offline requirements, privacy, sharing, scale and complexity.

---

# Part 6 — Unit-III Revision

## 23. Revision Map

| Topic | Recall points |
|---|---|
| Advanced layouts | `LinearLayout`, `RelativeLayout`, `ConstraintLayout`, responsive design |
| Android widgets | `TextView`, `EditText`, `Button`, `ImageView`, `CheckBox`, `RadioButton`, `Switch`, `Spinner`, `ProgressBar` |
| RecyclerView | `RecyclerView`, Adapter, `ViewHolder`, `LayoutManager`, data binding and recycling |
| Data persistence | Why apps save data; local and cloud storage choices |
| SQLite | Tables, rows, columns, `SQLiteOpenHelper`, CRUD |
| Content Providers | Provider, `ContentResolver`, content URI and permissions |
| Cursor | Navigation, reading records and resource management |
| Firebase | Realtime Database, cloud storage, synchronization, rules and access control |

## 24. Important Differences

### SharedPreferences vs SQLite
- `SharedPreferences`: small key-value settings.
- SQLite: structured records organized into tables and queried with SQL.

### SQLite vs Firebase Realtime Database
- SQLite stores structured data locally on the device.
- Firebase Realtime Database stores cloud data and supports synchronization for authorized clients.

### ContentResolver vs Content Provider
- A Content Provider exposes data through a standard interface.
- A ContentResolver is the client API used to interact with providers.

### Adapter vs ViewHolder
- Adapter creates item views and binds data.
- ViewHolder stores references to the views in one item.

### CheckBox vs RadioButton
- CheckBox supports multiple independent selections.
- RadioButton within a RadioGroup is commonly used for one choice from a set.

---

# Part 7 — Practice Questions

## Short-answer questions

1. Define data persistence.
2. Name four Android storage mechanisms.
3. What is SharedPreferences used for?
4. Define a database table, row and column.
5. What is the purpose of `SQLiteOpenHelper`?
6. Expand CRUD.
7. What is a Content Provider?
8. What does a content URI identify?
9. What is a Cursor?
10. What does `moveToNext()` do?
11. What is Firebase Realtime Database?
12. State two security considerations for a cloud database.

## Descriptive questions

1. Compare Android storage mechanisms and explain when each may be appropriate.
2. Explain the database creation flow using `SQLiteOpenHelper`.
3. Explain CRUD operations with examples.
4. Describe Content Provider architecture and the role of ContentResolver.
5. Explain cursor-based retrieval and list cursor best practices.
6. Explain the architecture and synchronization concept of Firebase Realtime Database.
7. Compare local and cloud storage.
8. Discuss offline/online behaviour, data consistency and security.
9. Explain how RecyclerView, data persistence and a database can work together in a student-record app.

## Viva & Interview Questions

1. Why should a password not be stored as plain SharedPreferences data?
2. Why should SQL selection arguments be used?
3. What is the difference between `onCreate()` and `onUpgrade()` in `SQLiteOpenHelper`?
4. Why must a Cursor be closed?
5. Does knowing a content URI automatically grant access to its data?
6. Why might an app use both local storage and cloud storage?
7. What can happen when two devices update the same record?
8. Why are Firebase security rules important?
9. What should an app do if a network request fails?
10. How would you choose storage for an offline-first notes app?

---

# Final Checklist

- [ ] I can explain data persistence and compare storage options.
- [ ] I can explain SharedPreferences and local storage.
- [ ] I understand tables, rows, columns and primary keys.
- [ ] I can describe `SQLiteOpenHelper` and CRUD.
- [ ] I can read query results with a Cursor and close it safely.
- [ ] I can explain Content Provider, ContentResolver, URI and permissions.
- [ ] I can describe Firebase Realtime Database and synchronization.
- [ ] I understand offline/online, consistency and security concerns.
- [ ] I can revise Unit-III layouts, widgets and RecyclerView concepts.

## Key Takeaway

**Choose storage according to the problem:** use preferences for small settings, files for file content, SQLite for local structured records, Content Providers for controlled data access across app boundaries, and a cloud database when shared or synchronized data is required. Design access, failure handling and security alongside the storage choice.
