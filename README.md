# 🤖 AI Agent for Formula Student Rules (LightRAG + n8n)

This project demonstrates an intelligent AI agent capable of answering complex and relational questions about the Formula Student Germany (FSG) rules. It leverages a **Graph RAG** approach using **LightRAG** to build a structured knowledge base from the official PDF rulebook, and **n8n** to orchestrate the AI agent's interaction with the user and the knowledge base.

The primary goal is to provide precise, context-aware answers, especially for questions involving intricate relationships between different rules and concepts within the highly structured FSG rulebook.

## 🌟 Why Graph RAG? (Naive RAG vs. Graph RAG)

For complex, interlinked documents like rulebooks, a "Naive RAG" (simple chunk-based retrieval) often falls short. A single rule (e.g., `T14.5.8`) might reference another rule (`A6.6`), which in turn depends on a safety standard (`T13.1`). A Naive RAG struggles to follow these complex, multi-hop connections.

This project employs a **Graph RAG** approach to overcome these limitations. LightRAG extracts entities (key concepts) and their relationships, forming a navigable knowledge graph.

| Feature | Naive RAG (Simple Chunks) | Graph RAG (LightRAG) |
| :--- | :--- | :--- |
| **How it Works** | Finds and "stuffs" raw text chunks based on keyword matching. | Finds precise concepts (`entities`) and their `relationships` in a structured graph. |
| **Context Quality** | Context can be "noisy," redundant, and include irrelevant paragraphs. | Context is precise, clean, and explicitly shows *how* concepts are linked. |
| **Query Type** | Good for simple, fact-based definitions ("What is a TSMS?"). | **Excellent** for complex, relational, and multi-hop questions ("What is the relationship between the ASMS and the shutdown circuit?"). |
| **Retrieval Mechanism** | `User Query` ➔ `Vector Search` ➔ `Top K Chunks` | `User Query` ➔ `LLM-Guided Graph Search` ➔ `Relevant Entities + Relationships + Source Chunks` |
| **Answer Quality** | Often requires the LLM to synthesize information from disjointed text, prone to less precise answers or "context stuffing." | Provides highly relevant, interconnected context, allowing the LLM to generate more accurate, concise, and nuanced answers. |
| **Robustness** | Can struggle with implied connections or rules spanning multiple sections. | Excels at understanding the *structure* of the rules, leading to more robust answers for complex scenarios. |

By using LightRAG, this agent can understand the *structure* of the rules, not just the words, leading to far more accurate and insightful answers.

## 🤖 Tech Stack

* **AI Agent Orchestration:** [n8n](https://n8n.io/)
* **Knowledge Base & Graph RAG:** [LightRAG](https://github.com/YourLightRAGRepoLinkHere) (Ensure this link is correct for your LightRAG setup if it's a fork or custom version)
* **Large Language Model (LLM):** OpenAI `gpt-4o-mini`
* **Embedding Model:** OpenAI `text-embedding-3-small`
* **Containerization:** Docker & Docker Compose

## 🏗️ Architecture

This project implements a Retrieval-Augmented Generation (RAG) pipeline designed for efficiency and accuracy:

1.  **User Interaction (n8n):** A user submits a query through the n8n chat interface.
2.  **AI Agent (n8n):** The n8n AI Agent receives the query and acts as the orchestrator.
3.  **Context Retrieval (LightRAG):** The n8n Agent calls the LightRAG server's `/query/data` endpoint via an HTTP Request node. Crucially, it sends the `query` along with `"only_need_context": true`.
4.  **Graph Search (LightRAG):** LightRAG uses its internal LLM to understand the query, performs a sophisticated graph search across its knowledge base (built from the FSG rules PDF), and retrieves relevant entities, relationships, and supporting text chunks.
5.  **Context Return (LightRAG):** Because of `"only_need_context": true`, LightRAG returns a clean JSON object containing only the retrieved context (entities, relationships, and chunks) **without** making an expensive LLM call for generation. This significantly reduces costs.
6.  **Answer Generation (n8n):** The n8n AI Agent receives this structured context. It then utilizes its own integrated LLM (`gpt-4o-mini`) to synthesize a human-readable and accurate answer based on the provided context.

## 🚀 Getting Started

### Prerequisites

Before you begin, ensure you have the following installed and running:

* **Docker Desktop:** Essential for running the LightRAG server.
* **n8n:** A self-hosted or cloud instance of n8n where you can import and run the workflow.
* **OpenAI API Key:** Required for both LightRAG's embeddings and n8n's LLM generation.

### 1. Set up LightRAG

1.  **Clone the Repository:**
    ```bash
    git clone [https://github.com/your-username/your-repo-name.git](https://github.com/your-username/your-repo-name.git)
    cd your-repo-name
    ```
2.  **Create Environment File:** Copy the example environment file and create your `.env` file.
    ```bash
    cp .env.example .env
    ```
3.  **Configure API Key:** Open the newly created `.env` file and replace `sk-YOUR_KEY_HERE` with your actual OpenAI API Key.
    ```ini
    # .env
    OPENAI_API_KEY=sk-YOUR_KEY_HERE
    LLM_BINDING=openai
    LLM_MODEL=gpt-4o-mini
    EMBEDDING_BINDING=openai
    EMBEDDING_MODEL=text-embedding-3-small
    # ... other LightRAG settings
    ```
4.  **Start LightRAG Server:**
    ```bash
    docker compose up -d
    ```
    This will start the LightRAG server in detached mode.

### 2. Set up n8n Workflow

1.  **Import Workflow:**
    * Open your n8n instance.
    * Navigate to "Workflows" and click "New".
    * Click the "Import" button (often a cloud icon with an arrow).
    * Select "From File" and upload the `workflow.json` file from this repository.
2.  **Configure HTTP Request Node:**
    * Locate the "HTTP Request" node within the imported workflow.
    * Ensure the URL in this node points to your LightRAG server.
        * If n8n is running on the same machine as Docker, use `http://host.docker.internal:9621/query/data`.
        * If n8n is on a different machine or cloud, replace `host.docker.internal` with the actual IP address or hostname where your LightRAG Docker container is accessible.
3.  **Activate Workflow:** Save and activate the n8n workflow.

### 3. Ingest Formula Student Rules PDF

1.  **Access LightRAG UI:** Open your web browser and go to `http://localhost:9621`.
2.  **Upload Document:** Navigate to the "Documents" tab.
3.  **Upload PDF:** Click "Upload Document" and select your Formula Student rules PDF (e.g., `FS-Rules_2025_v1.1.pdf`).
4.  **Monitor Ingestion:** LightRAG will now process the PDF, extract entities and relationships, and build its knowledge graph. Monitor the status until it shows "Completed".

## ⚡ How to Use

Once both LightRAG is running and ingested the document, and your n8n workflow is active, you can interact with the AI agent:

1.  Open the chat interface within your n8n workflow.
2.  Start asking questions about the Formula Student rules!

### Example Questions:

* "What is the rule for the Tractive System Active Light (TSAL)?"
* "Explain the relationship between the Safety Decoupling Controller (SDC) and the Autonomous System Brake (ASB)."
* "During an autonomous Trackdrive mission, if the perception system fails and the RES operator presses the emergency button, what is the required state of the AS-Status lights and why?"
* "Under what conditions is a kill switch required for the Tractive System?"

## 📸 Photos & Screenshots

Here are some visual aids to help understand the project setup and functionality.

### n8n Workflow Overview
A screenshot of the n8n workflow, showing both AI Agents the Naive Rag and LightGraph Rag one, and how they connect.

![n8n Workflow Screenshot- Naive Rag](images/Formula%20Student%20-%20Naive%20Rag.png)

![n8n Workflow Screenshot- LightGraph Rag](images/Formula%20Student%20-%20LightGraph%20Rag.png)

### LightRAG Knowledge Graph Visualization
A screenshot from the LightRAG UI showing a portion of the generated knowledge graph, highlighting entities and relationships.

![LightRAG Graph Visualization](images/lightrag_graph_visualization.gif)


## ⚙️ Key Configuration & Cost Optimization

* **LightRAG `.env`:** The `LLM_MODEL=gpt-4o-mini` and `EMBEDDING_MODEL=text-embedding-3-small` define the underlying AI models.
* **Cost Efficiency:** The n8n workflow explicitly sets `"only_need_context": true` in its HTTP request to LightRAG. This ensures that LightRAG only performs the efficient retrieval step (using embedding models) and **avoids a redundant, expensive LLM generation call** on its end. The main LLM for generating the final answer is handled once by the n8n AI Agent.

---
