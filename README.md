
# Enterprise-Grade Serverless Chatbot with Ontology-Guided Graph RAG

An advanced, production-ready Generative AI agent architecture designed to enforce strict domain knowledge constraints, eliminate LLM hallucinations, and optimize serverless infrastructure costs. This repository outlines the high-level software engineering decisions, stateful routing logic, and cloud-native serverless blueprints for building resilient conversational AI workflows using LangGraph and AWS Serverless layers.

> **Notice**: The source code of this repository is private and proprietary to protect core intellectual property and business logic. This document serves as a high-level architectural blueprint and technical overview of the production system.

---

## Key Architectural Highlights

- **Ontology-Guided State Management**: Implemented with **LangGraph** to maintain robust conversation history across multi-turn interactions, restricting LLM generation pathways within predefined structural knowledge taxonomies to guarantee deterministic outputs.
- **Context-Aware Query Expansion**: Features an ontology-driven expansion engine that maps raw user contextual queries to specialized domain ontologies at the code level, significantly boosting semantic retrieval recall.
- **Enterprise-Grade Security Layer**: Integrates client/server-side HTML sanitization (XSS mitigation) and regex-driven PII redaction (masking sensitive identifiers) to secure user payloads prior to LLM routing.
- **100% Free Serverless Focus**: Fully containerized via **Docker** and deployed on **AWS Lambda, ECR, and API Gateway**, completely eliminating infrastructure scaling overhead and GPU idle costs while optimizing cold-starts to under 2 seconds.

---

## System Workflow & Routing Architecture

The agent governs the conversation flow through a strict state machine, employing conditional routing to ensure maximum response precision while optimizing token budget:

```text
       [User Input Query]
               │
               ▼
   [Security Layer: XSS & PII Masking]
               │
               ▼
  [Ontology-Driven Query Expansion]
               │
       ┌───────┴───────┐
       ▼               ▼
[Pinecone Serverless] [External Research APIs]
(Vector DB Search)    (Real-time Literature)
       │               │
       └───────┬───────┘
               ▼
     [Retrieval Grader Node] ─── (Evaluates Utility via Pydantic Schema)
               │
               ▼
       [Conditional Router]
        /      │       \
    (Yes)     (No)    (Max Loop Exceeded / Fallback)
      /        │         \
     ▼         ▼          ▼
[Generator] [Query Rewriter] [Fallback Generator]
     │         │              │
   [END]       └──────────────┘ (Increments Loop Count & Re-executes)

Core Architectural Decisions & "The Why"

1. Why Ontology-Guided Graph RAG over Standard RAG?
Standard RAG pipelines often fail when the initial vector database search retrieves sub-optimal or irrelevant documents, leading to high hallucination rates. By introducing an Ontology-Guided pattern via LangGraph, the system acts as its own structural judge. It maps user query intents to a deterministic knowledge graph taxonomy at the code level, ensuring that the model stays grounded within authorized domain constraints and validated academic literature.

2. Why Containerized AWS Lambda over Dedicated GPU Hosting?
Hosting open-source LLMs or complex application layers on raw cloud GPU instances introduces significant Ops overhead, complex scaling policies, and high idle costs. This architecture utilizes Dockerized AWS Lambda (configured at 2GB memory) combined with managed AI endpoints to ensure 100% operational excellence and zero idle cost. Cloud-native race conditions are programmatically mitigated using AWS CLI waiters, driving cold-start latencies to under 2 seconds.

Production-Ready Technical Stack (AWS Blueprint)
* Orchestration: LangGraph / LangChain Core
* Vector Storage: Pinecone Serverless (Indexed with custom ontology metadata tags)
* Data Fetching: External Research Repository APIs (Real-time academic literature grounding)
* Execution & Ingestion: AWS Lambda, AWS ECR, AWS API Gateway (HTTP API)
* Security: Regex-driven Automated PII Masking, HTML Sanitizers

Conceptual Code Structure (Implementation Pattern)
Below is the abstract design pattern showing how the state machine and structured evaluation objects are coupled to maintain a production-grade application state:
Python

# Conceptual layout illustrating the decoupling of State and Evaluation Schemas
from pydantic import BaseModel
from typing import List, Dict, Any

class AgentState(BaseModel):
    messages: List[Any]          # Multi-turn conversation logs
    ontology_metadata: Dict      # Domain-specific structural taxonomy tags
    retrieved_context: List      # Merged Vector DB & External API cache
    loop_count: int              # Anti-infinite loop safety switch

class EvaluationSchema(BaseModel):
    binary_score: str            # Strict 'yes' or 'no' constraints via regex
    reasoning_trace: str         # Algorithmic reasoning trace for deterministic logging

Contact & Collaboration
For inquiries regarding the technical architecture, benchmarking results, or system capabilities, please reach me out. 
