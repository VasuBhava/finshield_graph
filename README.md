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
