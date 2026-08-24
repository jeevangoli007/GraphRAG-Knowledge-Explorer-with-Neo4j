## Module 6 – Advanced RAG & GraphRAG
# GraphRAG-Knowledge-Explorer-with-Neo4j

This project implements a **GraphRAG Knowledge Explorer** that combines a Neo4j knowledge graph with vector similarity search to answer questions that require reasoning across multiple connected facts.

The system extracts entities and relationships from a document using an Ollama LLM, stores the extracted knowledge in Neo4j, performs graph traversal and vector retrieval, combines the results using Reciprocal Rank Fusion (RRF), and generates the final answer using an LLM.

---

## 1. Project Objective

The assignment requires building a GraphRAG system that can:

1. Extract entities from a knowledge-rich document.
2. Extract relationships between entities using an LLM.
3. Convert the extracted information into knowledge-graph triples.
4. Store entities and relationships in Neo4j.
5. Perform vector similarity retrieval.
6. Extract important entities from user questions.
7. Traverse the Neo4j graph for 1–2 hops.
8. Combine graph retrieval and vector retrieval using RRF.
9. Answer multi-hop questions.
10. Compare traditional Vector RAG with GraphRAG.

---

## 2. Architecture

```text
                    KNOWLEDGE DOCUMENT
                           |
                           v
                       Chunking
                           |
                           v
                    Ollama / Qwen LLM
                           |
             +-------------+-------------+
             |                           |
             v                           v
          Entities                 Relationships
             |                           |
             +-------------+-------------+
                           |
                           v
                    Knowledge Triples
              (Entity A, Relation, Entity B)
                           |
                           v
                    +-------------+
                    |    Neo4j    |
                    | Knowledge   |
                    |    Graph    |
                    +------+------+
                           |
              +------------+------------+
              |                         |
              v                         v
       Graph Retrieval          Vector Retrieval
              |                         |
              |                    FAISS Search
              |                         |
              +------------+------------+
                           |
                           v
                 Reciprocal Rank Fusion
                           |
                           v
                    Hybrid Context
                           |
                           v
                       Ollama LLM
                           |
                           v
                    Final Answer
```

---

## 3. Technology Stack

| Technology | Purpose |
|---|---|
| Python | Main programming language |
| Jupyter Notebook | Development and demonstration |
| Ollama | Local LLM and embeddings |
| Qwen | Entity extraction and answer generation |
| Neo4j | Knowledge graph database |
| LangChain | LLM and retrieval orchestration |
| FAISS | Vector similarity search |
| Pydantic | Structured LLM output |
| Pandas | Evaluation and result tables |

---

## 4. Project Structure

```text
GraphRAG-Knowledge-Explorer/
│
├── data/
│   └── document.txt
│
├── GraphRAG_Knowledge_Explorer_Neo4j.ipynb
│
├── graphrag_comparison.csv
│
└── README.md
```

The notebook automatically creates `data/document.txt` if it does not already exist.

---

## 5. Requirements

### Python

Recommended:

```text
Python 3.10+
```

### Ollama

Install Ollama from:

https://ollama.com/

After installation, verify:

```bash
ollama --version
```

Pull the models:

```bash
ollama pull qwen3:1.7b
ollama pull nomic-embed-text
```

Verify:

```bash
ollama list
```

You should see the installed models.

---

## 6. Neo4j Installation

You can use either:

### Option A – Neo4j Desktop / Community Edition

Install Neo4j locally and create a database.

Typical local connection:

```text
URI: bolt://localhost:7687
Username: neo4j
Password: your_password
```

### Option B – Neo4j Aura

Create a free Neo4j Aura database and use the connection URI and password supplied by Neo4j.

For Aura, set:

```text
NEO4J_URI
NEO4J_USERNAME
NEO4J_PASSWORD
```

---

## 7. Install Python Dependencies

Open a terminal or run the installation cell inside Jupyter:

```bash
pip install -U neo4j langchain langchain-community langchain-core langchain-ollama langchain-text-splitters faiss-cpu pydantic pandas tabulate
```

---

## 8. Start Ollama

Make sure Ollama is running.

Check installed models:

```bash
ollama list
```

If necessary:

```bash
ollama pull qwen3:1.7b
ollama pull nomic-embed-text
```

The notebook uses:

```text
Chat model:
qwen3:1.7b

Embedding model:
nomic-embed-text
```

If you use different models, edit these variables in the notebook:

```python
CHAT_MODEL = "qwen3:1.7b"
EMBED_MODEL = "nomic-embed-text"
```

---

## 9. Configure Neo4j

The notebook defaults to:

```python
NEO4J_URI = "bolt://localhost:7687"
NEO4J_USERNAME = "neo4j"
NEO4J_PASSWORD = "password"
```

Change the password to your Neo4j database password.

For example:

```python
NEO4J_PASSWORD = "MyNeo4jPassword"
```

For security, environment variables can also be used:

```bash
set NEO4J_URI=bolt://localhost:7687
set NEO4J_USERNAME=neo4j
set NEO4J_PASSWORD=your_password
```

---

## 10. Running the Project

Start:

1. Ollama
2. Neo4j
3. Jupyter Notebook

Then open:

```text
GraphRAG_Knowledge_Explorer_Neo4j.ipynb
```

Run the notebook cells in order.

The main pipeline is:

```text
Load document
      ↓
Chunk document
      ↓
Extract entities and relationships
      ↓
Create Neo4j graph
      ↓
Create FAISS vector index
      ↓
Retrieve graph context
      ↓
Retrieve vector context
      ↓
RRF hybrid retrieval
      ↓
Generate answer
```

---

## 11. Knowledge Graph Extraction

The LLM extracts four types of entities:

```text
Person
Organization
Location
Concept
```

Example:

```text
Elon Musk
Tesla
Austin
Electric Vehicles
```

Relationships can include:

```text
CEO_OF
FOUNDED
COFOUNDED
HEADQUARTERED_IN
ACQUIRED
REBRANDED_AS
OPERATES
ASSOCIATED_WITH
```

The exact relationships depend on the document.

---

## 12. Triple Representation

The extracted graph is represented using triples:

```text
(Entity A, Relationship, Entity B)
```

Example:

```text
(Elon Musk, CEO_OF, Tesla)

(Tesla, HEADQUARTERED_IN, Austin)

(Elon Musk, FOUNDED, SpaceX)

(SpaceX, HEADQUARTERED_IN, Hawthorne)
```

These triples are loaded into Neo4j as graph nodes and edges.

---

## 13. Neo4j Graph

The notebook creates nodes such as:

```text
(:Entity {name: "Elon Musk", type: "PERSON"})
(:Entity {name: "Tesla", type: "ORGANIZATION"})
(:Entity {name: "Austin", type: "LOCATION"})
```

And relationships such as:

```text
(Elon Musk)-[:CEO_OF]->(Tesla)

(Tesla)-[:HEADQUARTERED_IN]->(Austin)
```

---

## 14. Verify the Graph in Neo4j Browser

Open Neo4j Browser and run:

```cypher
MATCH (n)-[r]->(m)
RETURN n, r, m
```

To view Elon Musk's 1–2 hop neighborhood:

```cypher
MATCH p=(n:Entity {name: 'Elon Musk'})-[*1..2]-(m)
RETURN p
```

Count entities:

```cypher
MATCH (n:Entity)
RETURN count(n) AS entities
```

Count relationships:

```cypher
MATCH ()-[r]->()
RETURN count(r) AS relationships
```

Show organizations:

```cypher
MATCH (n:Entity)
WHERE n.type = 'ORGANIZATION'
RETURN n.name
ORDER BY n.name
```

---

## 15. Vector Retrieval

The project creates a FAISS vector index from the document chunks.

```python
vector_store = FAISS.from_texts(
    chunks,
    embedding=embeddings
)
```

A user question is converted into an embedding and compared with document chunk embeddings.

Example:

```text
Question:
Where is Tesla headquartered?

        ↓

Embedding similarity

        ↓

Relevant document chunks
```

---

## 16. Graph Retrieval

Graph retrieval works differently.

For a query such as:

```text
Which organization was founded by the person who is the CEO of Tesla?
```

The system:

```text
Question
   ↓
Extract Tesla
   ↓
Find Tesla in Neo4j
   ↓
Traverse connected relationships
   ↓
Find Elon Musk
   ↓
Follow founder relationships
   ↓
Return connected organizations
```

This allows the system to reason across multiple graph hops.

---

## 17. Reciprocal Rank Fusion

The project combines vector and graph results using RRF.

Formula:

```text
RRF(d) = Σ 1 / (k + rank)
```

A result appearing near the top of both retrieval systems receives a stronger combined score.

This produces a hybrid context containing:

```text
Vector evidence
+
Graph evidence
```

---

## 18. GraphRAG vs Vector RAG

### Vector RAG

```text
Question
   ↓
Vector similarity
   ↓
Relevant text chunks
   ↓
LLM
   ↓
Answer
```

Vector RAG is effective when the answer is directly contained in a semantically similar passage.

### GraphRAG

```text
Question
   ↓
Entity extraction
   ↓
Neo4j graph traversal
   +
Vector similarity
   ↓
RRF
   ↓
Hybrid context
   ↓
LLM
   ↓
Answer
```

GraphRAG is especially useful when the answer requires multiple connected facts.

---

## 19. Multi-Hop Questions

The notebook tests at least five questions.

Examples:

### Question 1

```text
Which company is headquartered in the same city as Tesla,
and who is a co-founder of that company?
```

### Question 2

```text
Which organization was founded by the person who is the CEO
of Tesla, and where is that organization headquartered?
```

### Question 3

```text
Which company did Tesla acquire, where was that company
headquartered, and who founded it?
```

### Question 4

```text
Which company is connected to Elon Musk through an acquisition
and was later rebranded, and what was its original name?
```

### Question 5

```text
Which organization is headquartered in Hawthorne,
who founded it, and what concept is associated with
one of its major systems?
```

---

## 20. Evaluation

The notebook generates a comparison table containing:

```text
Question
Vector RAG Answer
GraphRAG Answer
```

The evaluation template allows manual scoring.

Recommended scoring:

```text
0 = Incorrect
1 = Correct
```

Evaluate:

```text
Vector RAG Correctness
GraphRAG Correctness

Vector RAG Multi-Hop Reasoning
GraphRAG Multi-Hop Reasoning
```

---

## 21. Expected Result

A typical result should show that Vector RAG performs well for questions where the required information exists in one or two highly similar passages.

GraphRAG should have an advantage when the answer requires following relationships such as:

```text
Person
   ↓
Organization
   ↓
Location
   ↓
Another Organization
   ↓
Founder
```

The exact scores depend on the LLM, document, extraction quality, graph quality, and questions.

Do not claim that GraphRAG is always better. The purpose of the experiment is to demonstrate where explicit graph relationships help.

---

## 22. Troubleshooting

### Ollama connection error

Check:

```bash
ollama list
```

Make sure Ollama is running.

Try:

```bash
ollama run qwen3:1.7b
```

---

### Embedding model not found

Run:

```bash
ollama pull nomic-embed-text
```

Then verify:

```bash
ollama list
```

---

### Neo4j connection error

Check:

```text
NEO4J_URI
NEO4J_USERNAME
NEO4J_PASSWORD
```

For local Neo4j, verify that the database is running and Bolt is enabled.

---

### Authentication failed

Update:

```python
NEO4J_PASSWORD = "your_actual_password"
```

---

### Structured output error

If your local Qwen model does not support the required structured-output behavior reliably, use a newer/larger Ollama model and update:

```python
CHAT_MODEL = "your_model_name"
```

A model with stronger instruction following may produce more reliable graph extraction.

---

### FAISS installation problem

Try:

```bash
pip install -U faiss-cpu
```

Restart the Jupyter kernel afterward.

---

### Graph has too few relationships

Inspect the extraction output before Neo4j insertion:

```python
print(test_extraction.model_dump_json(indent=2))
```

If the LLM extracts too few facts:
- use a richer document,
- increase chunk size,
- use a stronger model,
- improve the extraction prompt.

---

## 23. Important Academic Note

The graph quality depends on the quality of the LLM extraction.

If the source document does not explicitly state a relationship, the extraction prompt instructs the model not to invent it.

This is important because GraphRAG should retrieve evidence from the source rather than create unsupported relationships.

---

## 24. Assignment Deliverables

Submit:

```text
1. GraphRAG_Knowledge_Explorer_Neo4j.ipynb
2. README.md
3. data/document.txt
4. graphrag_comparison.csv
```

If your course requires a GitHub repository, recommended structure:

```text
GraphRAG-Knowledge-Explorer/
│
├── data/
│   └── document.txt
│
├── GraphRAG_Knowledge_Explorer_Neo4j.ipynb
│
├── graphrag_comparison.csv
│
└── README.md
```

---

## 25. Viva / Interview Explanation

### What is GraphRAG?

GraphRAG combines retrieval-augmented generation with a knowledge graph. Instead of relying only on vector similarity, it retrieves connected entities and relationships from a graph.

### Why Neo4j?

Neo4j is a graph database designed to represent entities as nodes and relationships as edges. It supports Cypher queries and graph traversal.

### Why use vector search too?

Vector search is good at finding semantically relevant passages. Graph search is good at following explicit relationships. Combining both provides complementary evidence.

### What is a multi-hop question?

A multi-hop question requires more than one connected fact.

Example:

```text
Who founded the organization headquartered
in the city where another company is located?
```

The system may need:

```text
Company
   ↓
HEADQUARTERED_IN
   ↓
City
   ↓
HEADQUARTERED_IN
   ↓
Organization
   ↓
FOUNDED_BY
   ↓
Person
```

### What is RRF?

Reciprocal Rank Fusion combines ranked results from multiple retrieval systems without requiring their raw scores to be directly comparable.

### Why can Vector RAG fail?

The information required to answer a question may be distributed across multiple chunks. The most similar chunk may contain only one part of the reasoning chain.

### Why can GraphRAG help?

The graph explicitly stores relationships, allowing the retriever to follow connected facts across multiple hops.

---

## 26. Conclusion

This project demonstrates an end-to-end GraphRAG pipeline using:

```text
Ollama
+
Qwen
+
Neo4j
+
FAISS
+
LangChain
+
Reciprocal Rank Fusion
```

The system extracts structured knowledge from documents, stores it in Neo4j, retrieves both graph and vector context, combines those results, and uses an LLM to answer multi-hop questions.

The main advantage of the approach is that it combines the semantic retrieval capabilities of vector search with the explicit relationship reasoning capabilities of a knowledge graph.
