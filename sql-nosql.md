# SQL vs NoSQL — DevOps Quick Notes

## SQL vs NoSQL

| SQL | NoSQL |
|---|---|
| Relational | Non-relational |
| Tables, rows, columns | Documents, key-value, etc. |
| Structured schema | Flexible schema |
| Uses SQL | Database-specific queries |
| MySQL, PostgreSQL | MongoDB, DynamoDB |

> **SQL = Tables | NoSQL = Flexible data**

---

## Database Types

### 1. Relational / SQL
- MySQL → `3306`
- PostgreSQL → `5432`
- SQL Server → `1433`
- Oracle → `1521`

**Know:** Tables, CRUD, Primary Key, Foreign Key, Index, Transactions, Backup/Restore.

### 2. Document / NoSQL
- MongoDB → `27017`

> **MongoDB = Document database**

### 3. Key-Value
- Redis → `6379`

> **Redis = In-memory key-value store / Cache**

### 4. Wide-Column
- Cassandra
- DynamoDB

> **Used for distributed, highly scalable workloads**

### 5. Graph
- Neo4j

> **Used when relationships are important**

---

# AWS Databases

| Service | Type | Remember |
|---|---|---|
| RDS | SQL | Managed relational DB |
| Aurora | SQL | AWS cloud-optimized relational DB |
| DynamoDB | NoSQL | Key-value/document DB |
| ElastiCache | Redis/Memcached | Caching |
| DocumentDB | NoSQL | Document database |

---

# DevOps Database Skills

Know how to:

- Deploy
- Connect
- Secure
- Troubleshoot
- Backup/Restore
- Monitor
- Replicate
- Configure HA

### Connection Details

```text
DB_HOST
DB_PORT
DB_NAME
DB_USER
DB_PASSWORD