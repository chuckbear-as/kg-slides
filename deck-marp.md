---
marp: true
theme: default
paginate: true
backgroundColor: #1a1a2e
color: #e0e0e0
style: |
  section {
    font-family: 'Helvetica Neue', Arial, sans-serif;
  }
  h1 {
    color: #00d4ff;
  }
  h2 {
    color: #00d4ff;
  }
  strong {
    color: #ff6b6b;
  }
  table {
    font-size: 0.7em;
  }
  th {
    background-color: #16213e;
    color: #00d4ff;
  }
  td {
    background-color: #0f3460;
  }
  blockquote {
    border-left: 4px solid #00d4ff;
    color: #b0b0b0;
    font-style: italic;
  }
  a {
    color: #00d4ff;
  }
---

# You Don't Need a Knowledge Graph
## You Already Have One

**The Semantic Layer as the Enterprise Knowledge Graph**

---

# The Problem

LLMs are great at understanding natural language.

They are **terrible** at:
- Being databases
- Doing math
- Remembering facts
- Statistical reasoning

> The goal is to make them use **actual facts from data** rather than statistical correlation and word associations.

---

# The Conventional Wisdom

### "You need a Knowledge Graph for AI"

Build a Neo4j database. ETL your warehouse into it.
Nodes, edges, Cypher queries. A whole new infrastructure layer.

### But ask yourself:
**Where do the relationships already live?**

In your relational database. Defined by foreign keys, joins, and business logic.

---

# Two Paths to the Same Answer

> "Who issued John Smith's insurance policy?"

---

# Path 1: Knowledge Graph + Neo4j

1. ETL relational data into Neo4j
2. Create nodes: `Agent`, `Policy`, `Customer`
3. Create edges: `ISSUED`, `COVERS`
4. LLM generates Cypher:

```cypher
MATCH (a:Agent)-[:ISSUED]->(p:Policy)
      -[:COVERS]->(c {name:"John Smith"})
RETURN a.name
```

**Cost:** Duplicate data store, ETL pipeline, new infrastructure

---

# Path 2: Semantic Layer + SQL

1. Relationships already exist in the warehouse
2. SML defines them: Policy → Agent via `agent_id`, Policy → Customer via `customer_id`
3. LLM reads SML, generates SQL:

```sql
SELECT a.name
FROM policy p
JOIN agent a ON p.agent_id = a.id
JOIN customer c ON p.customer_id = c.id
WHERE c.name = 'John Smith'
```

**Cost:** None. Data stays where it is.

---

# Side by Side

| | Knowledge Graph | Semantic Layer |
|---|---|---|
| **Relationships** | Explicit edges in graph DB | Join definitions in SML |
| **Data location** | Duplicated into Neo4j | Stays in warehouse |
| **Query language** | Cypher | SQL (generated) |
| **Extra infra** | Neo4j + ETL pipeline | None |
| **Data freshness** | Depends on ETL lag | Real-time |
| **Scales with** | Separate concern | Inherits warehouse |

---

# "But What About Complex Traversals?"

The classic Neo4j argument:

> "Find everyone within 3 degrees of John Smith"

That requires a complex Cypher query... **or three simple SQL queries.**

---

# LLMs Don't Need One Perfect Query

An LLM with agentic reasoning can decompose:

**Step 1:** "Who issued John Smith's policy?" → simple join → Agent ID

**Step 2:** "What other policies did that agent issue?" → simple query → Policy list

**Step 3:** "Who are the customers on those policies?" → simple query → Names

Three trivial, **auditable**, **governed** SQL queries.
Each informed by the semantic layer.

---

# Why Decomposition Wins

### Complex Cypher (one shot)
```cypher
MATCH (a)-[:ISSUED]->(p)-[:COVERS]->(c)
      -[:RELATED_TO*1..3]-(x)
WHERE c.name = 'John Smith'
RETURN DISTINCT x.name
```
- Single opaque operation
- If it's wrong, good luck debugging

### Multi-step SQL (agentic)
- Each step is **auditable**
- Each step is **simple SQL** — less likely to be wrong
- The semantic layer **governs every step**

---

# What Actually Causes LLM Errors?

The failure modes are:
- Hallucinating a column name that doesn't exist
- Joining on the wrong key
- Misunderstanding what a metric means
- Missing a filter (soft deletes, date ranges)

**The semantic layer addresses all of these.**

A graph database addresses **none of them.**
You can hallucinate Cypher just as easily as SQL.

---

# The Principle

## LLMs should be the natural language interface, not the computation engine.

---

# The Right Tool for Each Job

| Task | LLM Role | Delegate To |
|---|---|---|
| "15% of $4,230?" | Parse question | Calculator |
| "Who issued the policy?" | Understand intent | Semantic layer + SQL |
| "Is this fraudulent?" | Frame question | Bayesian model |
| "How are these related?" | Navigate & summarize | Multi-step SQL |
| "Last quarter's revenue?" | Present answer | Governed metrics |

The LLM is the **orchestrator**, not the source of truth.

---

# The Neo4j Case Gets Weaker Over Time

As LLMs improve at multi-step reasoning:

- The "complex query" advantage of graph DBs **erodes**
  LLMs decompose complex questions into simple ones

- The "schema understanding" advantage of semantic layers **grows**
  LLMs read SML and reason about relationships

- The data duplication cost of Neo4j **stays constant or worsens**

---

# When You Actually Need a Graph Database

- Data is **natively graph-shaped** (social networks, fraud rings)
- You need **graph algorithms** (PageRank, community detection, shortest path)
- You're integrating **unstructured sources** (extracted entities from documents)

For enterprise analytics against warehouse data?

**The semantic layer is the knowledge graph.**

---

# The Missed Marketing Opportunity

AtScale could be saying:

> "You don't need a knowledge graph. You already have one.
> It's your semantic layer."

Instead of treating knowledge graphs as a separate complementary technology.

---

# Summary

1. **The semantic layer already encodes relationship metadata** that a knowledge graph would duplicate

2. **LLMs + simple tools beat LLMs + complex tools** — decomposition into auditable steps wins

3. **Use LLMs for language, delegate everything else** — math, data, statistics, reasoning — to authoritative tools

4. **The quality of the system depends on tool quality**, not on making the LLM smarter at things it can't do

---

# Thank You

**The Semantic Layer as the Enterprise Knowledge Graph**

*LLMs for language. Real tools for real answers.*
