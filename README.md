# FinShield Graph — AML & Synthetic Fraud Ring Detection

[![CognoDB Cloud](https://img.shields.io/badge/Database-CognoDB%20Cloud-blue)](https://console.cognodb.com)
[![Backend](https://img.shields.io/badge/Backend-NestJS%20(Clean%20Architecture)-E0234E)](https://nestjs.com/)
[![Frontend](https://img.shields.io/badge/Frontend-Next.js%2014%20(Atomic%20Design)-black)](https://nextjs.org/)

FinShield Graph is an investigative graph intelligence engine designed to uncover complex financial crimes, synthetic identity clusters, and circular money-laundering rings in real time. It uses **CognoDB Cloud** (openCypher over Bolt protocol) as the graph database layer.

---

## 🔗 Deliverables & Demo Links

* **Hosted Live Application**: [https://finshield-graph.vercel.app](https://finshield-graph.vercel.app) *(Replace with your live URL)*
* **Walkthrough Screen Recording**: [https://www.loom.com/share/your-recording-id](https://www.loom.com/share/your-recording-id) *(Replace with your Loom/video URL)*
* **Repository**: [https://github.com/your-username/finshield-graph](https://github.com/your-username/finshield-graph)[cite: 1]

---

## 📋 Use Cases

FinShield Graph addresses patterns that are common in financial crime investigations:

* **Circular Money Laundering (Mule Rings & Layering)**: Uncovers multi-hop fund movements where money cycles through intermediary accounts ($A \to B \to C \to D \to A$) to obscure origins.
* **Synthetic Identity Clusters**: Detects seemingly unrelated accounts sharing physical locations, gateway IPs, or hardware device fingerprints.
* **High-Risk Smurfing Paths**: Identifies high-risk entity clusters connected via rapid chained transactions.

---

## 💡 Why a Graph Database?

Relational databases store entities in isolated, tabular rows and compute relationships at query time using index lookups and join operations. As path depths increase, relational join complexity scales exponentially, leading to performance bottlenecks.

| Operational Vector | Relational Database (SQL) | Graph Database (CognoDB / openCypher) |
| :--- | :--- | :--- |
| **Multi-Hop Traversal** | Requires heavy recursive Common Table Expressions (`WITH RECURSIVE`) and multi-level self-joins. | Expressed naturally using variable-length paths: `(:Account)-[:TRANSFERRED*2..5]->(:Account)`. |
| **Index-Free Adjacency** | Pointer resolution via secondary index scans on foreign key tables ($O(\log N)$ per join). | Direct node-to-node pointer dereferencing ($O(1)$ per edge traversal). |
| **Entity Resolution** | Multi-table joins across user, address, device, and login tables result in Cartesian join overhead. | Graph pattern matching directly identifies shared identity anchors. |
| **Schema Evolution** | `ALTER TABLE` locks, table migrations, and null-heavy join schemas. | Dynamic, typed property graphs allow extending entity types without schema locks. |

---

## 📐 Graph Data Model & Schema

### Schema Diagram


```

```
             +-----------------------------------+
             |              Device               |
             |-----------------------------------|
             | id: String (PK)                   |
             | fingerprint: String               |
             | ipAddress: String                 |
             +-----------------------------------+
                   ^                       ^
                   | [:ACCESSED_FROM]      | [:ACCESSED_FROM]
                   |                       |

```

+-----------------------------------+     +-----------------------------------+
|              Account              |     |              Account              |
|-----------------------------------|     |-----------------------------------|
| id: String (PK)                   |     | id: String (PK)                   |
| ownerName: String                 |     | ownerName: String                 |
| riskScore: Integer                |     | riskScore: Integer                |
| balance: Float                    |     | balance: Float                    |
+-----------------------------------+     +-----------------------------------+
|               |                         ^               |
|               |       [:TRANSFERRED]    |               |
|               +-------------------------+               |
|                 - txId: String                          |
|                 - amount: Float                         |
|                 - timestamp: DateTime                   |
|                                                         |
| [:REGISTERED_AT]                       [:REGISTERED_AT] |
v                                                         v
+---------------------------------------------------------------+

| Address |
| --- |
| id: String (PK) |
| street: String |
| city: String |
| postalCode: String |
| +---------------------------------------------------------------+ |

```

### Nodes[cite: 1]
* `(:Account)`: Financial profile holding balance, owner metadata, and computed risk score[cite: 1].
* `(:Device)`: Digital client endpoint with hardware fingerprint and IP address[cite: 1].
* `(:Address)`: Physical location corresponding to customer registration[cite: 1].

### Relationships[cite: 1]
* `(:Account)-[:TRANSFERRED { txId, amount, timestamp }]->(:Account)`: Directed fund transactions[cite: 1].
* `(:Account)-[:ACCESSED_FROM { lastUsed }]->(:Device)`: Device authentication telemetry[cite: 1].
* `(:Account)-[:REGISTERED_AT]->(:Address)`: KYC residency connection[cite: 1].

---

## 🔍 Key Cypher Queries Explained

All queries use parameterized inputs via the official `neo4j-driver` over Bolt protocol[cite: 1].

### 1. Multi-Hop Circular Mule Ring Detection (2 to 5 Hops)[cite: 1]
Identifies funds cycled across variable-depth networks to launder capital[cite: 1]:
```cypher
MATCH path = (origin:Account)-[:TRANSFERRED*2..5]->(origin:Account)
WITH origin, relationships(path) AS txs, nodes(path) AS ringNodes
RETURN origin.id AS originId,
       [n in ringNodes | { id: n.id, name: n.ownerName, risk: n.riskScore }] AS ringPath,
       reduce(total = 0, t IN txs | total + t.amount) AS totalVolume,
       length(path) AS hopCount

```

### 2. Multi-Hop Entity Resolution (Shared Identity Clusters)



Finds synthetic accounts masquerading under different names but sharing physical or digital properties:

```cypher
MATCH (a1:Account)-[:ACCESSED_FROM|REGISTERED_AT]->(shared)<-[:ACCESSED_FROM|REGISTERED_AT]-(a2:Account)
WHERE a1.id < a2.id
RETURN a1.id AS account1, 
       a1.ownerName AS name1, 
       a2.id AS account2, 
       a2.ownerName AS name2, 
       labels(shared)[0] AS sharedEntityType,
       shared.id AS sharedEntityId

```

---

## 🏛 Architecture Overview

* **Backend (`apps/api`)**: Built with **NestJS** using strict **Clean Architecture (DDD)** principles:
* `domain/`: Pure business entities and abstract repository contracts (`IFraudRepository`).
* `application/`: Independent Use Case interactors (`DetectCircularRingsUseCase`, `FindSharedIdentifiersUseCase`).
* `infrastructure/`: Neo4j driver connection to CognoDB Cloud and openCypher repository implementations.


* `presentation/`: REST API controllers and input validation DTOs.


* **Frontend (`apps/web`)**: Built with **Next.js 14 App Router (TypeScript)** using **Atomic Design System**:
* `atoms/`: Badges, buttons, typography, spinners.
* `molecules/`: Tabs, stat cards, property lists.
* `organisms/`: Dynamic SSR-safe 2D force-directed graph canvas (`react-force-graph-2d`), alert feeds.
* `templates/`: Full-viewport split investigation layout.



---

## 🚀 Setup & Run Instructions

### Prerequisites

* Node.js >= 18.x
* npm >= 9.x
* A free **CognoDB Cloud** account



### 1. Provision CognoDB Instance

1. Go to [https://console.cognodb.com/signup](https://www.google.com/search?q=https://console.cognodb.com/signup) and create an account.


2. Provision a free `c0` instance in your preferred region.


3. Copy your Bolt URI (`bolt+s://<instance-id>.databases.cognodb.cloud`) and generated password.



### 2. Configure Environment Variables

Create `.env` inside `apps/api/`:

```env
PORT=5000
COGNODB_URI=bolt+s://<your-instance-id>.databases.cognodb.cloud
COGNODB_USER=cognodb
COGNODB_PASSWORD=your_instance_password

```

Create `.env.local` inside `apps/web/`:

```env
NEXT_PUBLIC_API_BASE_URL=http://localhost:5000/api/fraud

```

### 3. Install Dependencies & Seed Database

```bash
# Install workspace dependencies from the root directory
npm install

# Run database seed script to populate realistic graph data into CognoDB
npm run seed --prefix apps/api

```

### 4. Start Development Servers

```bash
# Start backend API (NestJS - Port 5000)
npm run start:dev --prefix apps/api

# In a separate terminal, start frontend dashboard (Next.js - Port 3000)
npm run dev --prefix apps/web

```

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## 🖼 UI & Application Screenshots

| Interactive Graph Canvas | Ring Alert & Entity Resolution Feed |
| --- | --- |
| *(Add canvas screenshot here)*<br> | *(Add inspection sidebar screenshot here)*<br> |

---



