# 3-Tier Architecture

---

## 1. Desktop Application vs Web Application

### Desktop Application

A desktop application is installed and runs directly on the user's computer.

```text
User
 |
 v
Desktop Application
 |
 v
Local Storage / Database
```

### Challenges

* Requires installation on each machine
* Updates/upgrades need to be managed
* Uses local system resources
* Application may hang or crash
* Data may be stored locally
* If the local system fails, data recovery can be difficult
* Usually limited to the systems where the application is installed
* Centralized management can be difficult

---

## 2. Web-Based Application

A web application runs on servers and is accessed through a web browser.

```text
User
 |
 v
Browser
 |
 v
Web Application
 |
 v
Database
```

### Advantages

* No application installation required on every client
* Accessible through a browser
* Centralized application management
* Easier deployment and upgrades
* Server-side processing
* Centralized data storage
* Easier to scale
* Better control over security and access

---

# 3. Tier Architecture

A **tier** represents a logical separation of application responsibilities.

The common architecture is:

```text
Presentation / Web Tier
          |
          v
Application / Business Tier
          |
          v
Database / Data Tier
```

### Main Principle

```text
WEB → LOGIC → DATA
```

* Web Tier → Handles client requests
* Application Tier → Processes business logic
* Database Tier → Stores and retrieves data

---

# 4. 1-Tier Architecture

In a 1-tier architecture, all application components are located together.

```text
+----------------------------------+
|            SERVER                |
|                                  |
|  Web / UI                        |
|  Application Logic               |
|  Database                        |
+----------------------------------+
```

Everything runs on the same system.

### Example

A small application containing:

```text
Frontend
   +
Application Logic
   +
Database
```

on a single machine.

### Characteristics

* Simple architecture
* Easy to set up
* Suitable for small applications
* Limited scalability
* Single point of failure
* Difficult to scale individual components independently

### Memory

> **Everything together = 1-Tier**

---

# 5. 2-Tier Architecture

In a 2-tier architecture, the application is divided into two major components.

```text
Client / Application
        |
        v
     Database
```

The client/application directly communicates with the database.

### Example

```text
Application
     |
     | SQL
     v
   MySQL
```

The application may perform:

```text
INSERT
SELECT
UPDATE
DELETE
```

directly against the database.

### Characteristics

* Better separation than 1-tier
* Database is separated from the client/application
* Common in traditional client-server applications
* Application directly accesses the database
* Scaling and security can become difficult as the application grows

### Memory

> **Application → Database = 2-Tier**

---

# 6. 3-Tier Architecture

In 3-tier architecture, the application is divided into three separate tiers.

```text
User
 |
 v
Web Tier
 |
 v
Application Tier
 |
 v
Database Tier
```

The three tiers have different responsibilities.

```text
1. Web Tier
       ↓
2. Application Tier
       ↓
3. Database Tier
```

---

# 7. Professional Example — E-Commerce Application

Consider an online shopping application.

The user opens:

```text
https://shop.example.com
```

and searches for:

```text
Laptop
```

The request flows through the architecture.

```text
                         USER
                           |
                           v
                    +-------------+
                    |   WEB TIER  |
                    |             |
                    | ALB / Nginx |
                    +------+------+
                           |
                           v
                    +-------------+
                    | APPLICATION |
                    |    TIER     |
                    |             |
                    | Java/NodeJS |
                    +------+------+
                           |
                           v
                    +-------------+
                    |  DATABASE   |
                    |    TIER     |
                    |             |
                    | MySQL/RDS   |
                    +-------------+
```

---

# 8. Request Flow

Suppose the user searches:

```text
Laptop
```

## Step 1 — User

The user sends an HTTP/HTTPS request.

```text
GET /products?search=laptop
```

---

## Step 2 — Load Balancer

The request reaches the Load Balancer.

```text
User
 |
 v
Load Balancer
```

The Load Balancer distributes the request to an available Web Server.

```text
                 Load Balancer
                 /     |     \
                v      v      v
             Web-1  Web-2  Web-3
```

---

# 9. Web Tier

The Web Tier is responsible for handling the client-facing part of the application.

### Technologies

```text
Nginx
Apache
HAProxy
AWS ALB
```

### Frontend Technologies

```text
HTML
CSS
JavaScript
React
Angular
Vue
```

### Responsibilities

* Receive HTTP/HTTPS requests
* Serve static content
* SSL/TLS termination
* Request routing
* Load balancing
* Forward API requests to Application Servers

Example:

```text
Browser
   |
   v
Nginx
   |
   v
Application Server
```

### Important

The Web Tier should generally **not contain the main business logic**.

---

# 10. Application Tier

The Application Tier contains the application's **business logic**.

### Technologies

```text
Java
.NET
Python
NodeJS
C++
Go / Golang
```

Example:

```text
NodeJS Application Server
```

The Application Server receives:

```text
GET /products?search=laptop
```

It processes the request.

It may:

```text
1. Validate the request
2. Apply business rules
3. Authenticate the user
4. Query the database
5. Process the returned data
6. Send the response
```

---

# 11. CRUD

Application servers commonly perform CRUD operations against databases.

```text
C → Create
R → Read
U → Update
D → Delete
```

### Create

```sql
INSERT
```

### Read

```sql
SELECT
```

### Update

```sql
UPDATE
```

### Delete

```sql
DELETE
```

Example:

```text
Application Server
        |
        | SQL Query
        v
      MySQL
```

---

# 12. Database Tier

The Database Tier is responsible for storing and retrieving persistent application data.

### Relational Databases

```text
MySQL
PostgreSQL
Oracle
Microsoft SQL Server
```

### NoSQL Databases

```text
MongoDB
Cassandra
DynamoDB
```

### In-Memory Data Stores / Caches

```text
Redis
```

The Application Tier communicates with these systems.

```text
Application
     |
     v
Database / Data Store
```

---

# 13. Complete E-Commerce Example

Suppose a customer searches for:

```text
Laptop
```

The complete flow is:

```text
                         USER
                           |
                           | HTTPS
                           v
                    +-------------+
                    |    ALB      |
                    | Load Balancer|
                    +------+------+
                           |
                           v
                 +-------------------+
                 |     WEB TIER      |
                 |                   |
                 | Nginx             |
                 | HTML/CSS/JS       |
                 +---------+---------+
                           |
                           | API Request
                           v
                 +-------------------+
                 | APPLICATION TIER  |
                 |                   |
                 | NodeJS / Java     |
                 | Business Logic    |
                 +---------+---------+
                           |
                           | SQL / Query
                           v
                 +-------------------+
                 |    DATABASE TIER  |
                 |                   |
                 | MySQL / PostgreSQL|
                 +-------------------+
```

---

# 14. Response Flow

The response travels back in the opposite direction.

```text
Database
   |
   v
Application Server
   |
   v
Web Server
   |
   v
Load Balancer
   |
   v
User Browser
```

For example:

```text
MySQL
  ↓
Product information
  ↓
Application processes it
  ↓
Web server returns response
  ↓
Browser displays laptops
```

---

# 15. Why 3-Tier Architecture?

## Separation of Responsibilities

Each tier has a specific responsibility.

```text
Web Tier
   ↓
Presentation / Request Handling

Application Tier
   ↓
Business Logic

Database Tier
   ↓
Data Storage
```

This is called **separation of concerns**.

---

# 16. Loose Coupling / Decoupling

One of the important concepts in 3-tier architecture is **decoupling**.

The tiers should not be tightly dependent on each other.

For example:

```text
Web Tier
   |
   ↓
Application API
   |
   ↓
Database
```

The Web Tier communicates with the Application Tier through an interface/API instead of directly accessing the database.

### Bad Design

```text
Browser
   |
   +-----------> Database
   |
   +-----------> Application
```

### Better Design

```text
Browser
   |
   v
Web Tier
   |
   v
Application Tier
   |
   v
Database Tier
```

This provides better:

* Security
* Maintainability
* Scalability
* Flexibility

---

# 17. Independent Scaling

Each tier can be scaled independently.

Example:

```text
WEB TIER
3 servers

APPLICATION TIER
10 servers

DATABASE TIER
2 servers
```

If application traffic increases:

```text
Application Servers

3
↓
5
↓
10
```

You don't necessarily need to increase the Web Tier or Database Tier at the same rate.

---

# 18. High Availability

Multiple servers can be deployed in each tier.

Example:

```text
                    ALB
                 /   |   \
                /    |    \
               v     v     v
             Web1  Web2  Web3
               \     |     /
                \    |    /
                   App
                /   |   \
              App1 App2 App3
                 \   |   /
                    DB
```

If one Web Server fails:

```text
Web-1 ❌
```

the Load Balancer can send traffic to:

```text
Web-2
Web-3
```

---

# 19. Security

The Database Tier should normally be isolated from direct Internet access.

### Bad

```text
Internet
   |
   v
MySQL
```

### Better

```text
Internet
   |
   v
Load Balancer
   |
   v
Web Tier
   |
   v
Application Tier
   |
   v
Database Tier
```

Example AWS Security Groups:

```text
Internet
   |
   | 80 / 443
   v
Web SG
   |
   | Application Port
   v
Application SG
   |
   | 3306
   v
Database SG
```

For MySQL:

```text
TCP 3306
```

should normally be allowed only from the required Application Tier, not from the entire Internet.

---

# 20. AWS 3-Tier Architecture

A typical AWS implementation:

```text
                         INTERNET
                            |
                            v
                        Route 53
                            |
                            v
                   Application Load Balancer
                            |
              +-------------+-------------+
              |                           |
              v                           v
        Public Subnet               Public Subnet
              |                           |
           Web-1                       Web-2
           Nginx                       Nginx
              |                           |
              +-------------+-------------+
                            |
                            v
                    Private Subnets
                            |
                    Application Tier
                       EC2 / ECS / EKS
                       /      |      \
                      v       v       v
                    App-1   App-2   App-3
                       \      |      /
                        \     |     /
                            v
                    Database Subnets
                            |
                            v
                       Amazon RDS
                     MySQL/PostgreSQL
```

---

# 21. Where DevOps Fits

A DevOps Engineer may manage all parts of the architecture.

## Infrastructure

```text
AWS
VPC
Subnets
Route Tables
Security Groups
ALB
EC2
RDS
```

## Infrastructure as Code

```text
Terraform
```

## Web Tier

```text
Nginx
ALB
Route 53
TLS
```

## Application Tier

```text
Docker
Kubernetes
EKS
ECS
Java
NodeJS
Python
```

## CI/CD

```text
Git
GitHub
Jenkins
GitHub Actions
```

## Monitoring

```text
CloudWatch
Prometheus
Grafana
```

---

# 22. 1-Tier vs 2-Tier vs 3-Tier

| Architecture | Structure                    |
| ------------ | ---------------------------- |
| 1-Tier       | Everything together          |
| 2-Tier       | Application → Database       |
| 3-Tier       | Web → Application → Database |

### 1-Tier

```text
[ Web + Application + Database ]
```

### 2-Tier

```text
[ Application ]
       |
       v
[  Database  ]
```

### 3-Tier

```text
[ Web ]
   |
   v
[ Application ]
   |
   v
[ Database ]
```

---

# 23. Important Correction

Do not think:

```text
3-Tier = 3 Servers
```

This is incorrect.

3-tier means **3 logical responsibilities**.

You can have:

```text
Web Tier
5 servers

Application Tier
20 servers

Database Tier
2 servers
```

and it is still a 3-tier architecture.

---

# 24. Important Terminology

### Web Tier

Also called:

```text
Presentation Tier
Frontend Tier
Web Server Tier
```

### Application Tier

Also called:

```text
Business Tier
Backend Tier
Middleware Tier
Application Server Tier
```

### Database Tier

Also called:

```text
Data Tier
Database Layer
Data Store
```

---

# 25. One-Line Memory

```text
WEB → LOGIC → DATA
```

### Web Tier

> Receives and routes requests.

### Application Tier

> Processes business logic.

### Database Tier

> Stores and retrieves data.

---

# 26. Interview Answer

### What is 3-Tier Architecture?

> 3-tier architecture is a software architecture where an application is divided into three logical tiers: the Web/Presentation Tier, Application/Business Tier, and Database/Data Tier. The Web Tier handles client requests and presentation, the Application Tier contains business logic and processes requests, and the Database Tier stores and retrieves persistent data. This separation provides better security, scalability, maintainability, and independent scaling.

---

# Final Memory Diagram

```text
                         USER
                           |
                           v
                  +----------------+
                  |    WEB TIER    |
                  |                |
                  | ALB / Nginx    |
                  | HTML/CSS/JS    |
                  +-------+--------+
                          |
                          | API
                          v
                  +----------------+
                  | APPLICATION    |
                  |     TIER       |
                  |                |
                  | Java/Node/Python|
                  | Business Logic |
                  +-------+--------+
                          |
                          | SQL / Query
                          v
                  +----------------+
                  |  DATABASE TIER |
                  |                |
                  | MySQL/Postgres |
                  | MongoDB/Redis  |
                  +----------------+

              WEB → LOGIC → DATA
```

> **Remember:**
> **Web receives → Application processes → Database stores.**
