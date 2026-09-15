# SQLite

**SQLite3** is a lightweight, serverless, self-contained SQL database engine. Unlike traditional relational database management systems (RDBMS) like MySQL, PostgreSQL, or Microsoft SQL Server, SQLite does not run as a separate background server process. Instead, it is an embedded database library that integrates directly into the application running it.

---

### Key Architectural Characteristics

* **Serverless:** There is no client-server network protocol, daemon, or service to configure or manage. The database engine runs in the same memory space as the host application.
* **Single-File Storage:** An entire SQLite database—tables, indexes, triggers, schema, and data—is stored as a single, cross-platform ordinary disk file. Copying or backing up the database requires only copying that one file.
* **Zero Configuration:** It requires no administrative setup, user account creation, service initialization, or port configuration. Applications read and write directly to the database file via system I/O calls.
* **ACID Compliant:** Transactions are fully Atomic, Consistent, Isolated, and Durable, even across system crashes or sudden power outages.
* **Dynamic / Manifest Typing:** Unlike most SQL databases where column data types are strictly enforced, SQLite associates types with values rather than columns (with `INTEGER PRIMARY KEY` being a notable exception).

---

### Common Use Cases

* **Embedded Systems & Mobile Apps:** Default local data store on Android, iOS, Windows, macOS, smart TVs, and IoT devices.
* **Desktop Software & Browsers:** Used extensively for local storage (e.g., Google Chrome and Mozilla Firefox store bookmarks, history, and cookies in SQLite databases).
* **Application File Formats:** Many tools use SQLite as their native save-file format rather than inventing proprietary binary structures.
* **Testing & Prototyping:** Often used during software development to test database logic locally without spinning up Docker containers or dedicated database servers.

---

### SQLite vs. Client-Server Databases

| Feature | SQLite3 | Client-Server (PostgreSQL, MySQL) |
| --- | --- | --- |
| **Architecture** | Embedded library in application process | Standalone daemon listening on network ports |
| **Concurrency** | Multiple readers, but only one concurrent writer | High concurrent read/write throughput via fine-grained locking |
| **Network Access** | Local disk access only (no built-in network protocol) | Native network client-server protocol over TCP/IP |
| **Administration** | Zero setup; managed as a regular file | Requires user management, service tuning, and network firewalls |