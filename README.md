
# Enterprise-Grade Self-RAG Conversational Agent Architecture

An advanced, production-ready Generative AI agent architecture designed to mitigate LLM hallucinations and optimize infrastructure costs. This repository outlines the high-level structural design, routing logic, and cloud-native infrastructure blueprints for building resilient conversational AI workflows using LangGraph and Managed LLM Services.

> **Notice**: Due to proprietary business logic, commercial data privacy regulations, and strict confidentiality agreements, the underlying source code, exact prompt templates, and production dataset schemas are kept confidential. This documentation focuses exclusively on the core software engineering decisions, high-level architectural design, and system scalability patterns.

---

## Key Architectural Highlights

- **Deterministic State Management**: Implemented with **LangGraph** to maintain robust conversation history, user states, and tracking histories across multi-turn interactions.
- **Self-RAG (Self-Correction Loop)**: Features an automated retrieval grading mechanism that evaluates retrieved context utility before generation, preventing out-of-context or hallucinated responses.
- **No-Ops / Serverless Focus**: Designed to completely eliminate infrastructure scaling overhead, memory leaks, and GPU server maintenance by leveraging fully-managed computing layers.

---

## System Workflow & Routing Architecture

The agent governs the conversation flow through a strict state machine, employing conditional routing to ensure maximum response precision while optimizing token budget:

```text
       [User Input Query]
               │
               ▼
      [Context Retrieval] (Vector DB)
               │
               ▼
   ┌➔ [Retrieval Grader Node] ─── (Evaluates Document Relevance via Pydantic Schema)
   │           │
   │           ▼
   │   [Conditional Router]
   │    /      │       \
   │ (Yes)    (No)    (Max Loop Exceeded / Fallback)
   │  /        │         \
   ▼ ▼         ▼          ▼
[Generator] [Query Rewriter] [Fallback Generator] (Direct Response)
   │           │                  │
 [END]         └──────────────────┘ (Increments Loop Count & Re-executes)


Core Architectural Decisions & "The Why"

1. Why LangGraph over linear RAG?
Standard RAG pipelines often fail when the initial vector database search retrieves sub-optimal or irrelevant documents, leading to high hallucination rates. By introducing a Self-RAG pattern via LangGraph, the system acts as its own judge (LLM-as-a-Judge). It scores retrieved materials using structured outputs and dynamically decides whether to rewrite the search query or proceed to generation.

2. Why Managed Serverless Computing over Dedicated GPU Hosting?
Hosting open-source LLMs (e.g., Llama/Mistral) on raw cloud GPU instances (like AWS EC2 G5/P4) introduces significant Ops Overhead:
* Complex scaling policies for peak traffic.
* Vulnerability to runtime memory leaks.
* High idle costs during off-peak hours.
To ensure 100% operational excellence and zero idle cost, this architecture utilizes fully managed serverless AI endpoints. This approach decouples the computing/inference layer from internal systems, allowing the engineering team to focus entirely on application logic rather than infrastructure maintenance.
Production-Ready Technical Stack (Target AWS Blueprint)
While the business logic is decoupled, the system is designed to seamlessly port into the following cloud-native ecosystem for global scaling:
* Orchestration: LangGraph / LangChain Core
* Inference Layer: Amazon Bedrock (Utilizing Converse API for ultra-low latency & deterministic JSON structured outputs)
* Vector Storage: Managed Vector Database (Pinecone / Amazon OpenSearch Serverless)
* Execution & Microservices: AWS Lambda (Serverless Backend)
Conceptual Code Structure (Implementation Pattern)
Below is the abstract design pattern showing how the state machine and structured output objects are coupled to maintain a production-grade application state:
Python

# Conceptual layout illustrating the decoupling of State and Evaluation Schemas
class AgentState(BaseModel):
    messages: List[Any]          # Multi-turn conversation logs
    system_metadata: Dict        # Ephemeral contextual tags 
    retrieved_docs: List         # Context candidate cache
    loop_count: int              # Anti-infinite loop safety switch

class EvaluationSchema(BaseModel):
    binary_score: str            # Strict 'yes' or 'no' constraints via regex
    explanation: str             # Algorithmic reasoning trace

Contact & Collaboration
For inquiries regarding the technical architecture, benchmarking results, or system capabilities, please reach me out. 
