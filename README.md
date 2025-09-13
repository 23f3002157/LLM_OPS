# 🧠 LLMOps Chatbot

This repository implements a **full-lifecycle LLMOps pipeline** for a production-grade chatbot.  
It covers **development, experimentation, human-in-the-loop feedback, model registration, and deployment** using modern MLOps best practices.

---

## 🚀 Features

- ✅ **Guardrails for Safety** – Toxicity filtering with a Hugging Face **toxic-bert** model.  
- 🔍 **Retrieval-Augmented Generation (RAG)** – Document retrieval using **FAISS** for contextual responses.  
- 🤖 **Dynamic LLM Routing** – Intelligent **HeuristicRouter** selects the best model (e.g., GPT vs Claude 3.5) based on query context and feedback.  
- 📝 **Prompt Adaptation** – ControlHintAdapter dynamically adjusts system prompts based on feedback.  
- 📊 **Experimentation & Feedback** – Human-in-the-Loop (HITL) feedback loop with adaptive response tuning.  
- 📦 **MLflow Integration** – End-to-end model packaging, versioning, and registry management.  
- 🌐 **FastAPI Inference Server** – REST API with health and chat endpoints.  
- 🐳 **Docker + Kubernetes Ready** – Containerized and scalable deployment workflow.  

---

## 🧩 Core Components

### 1. Guardrail  
Ensures safe operation by filtering both user queries and LLM responses for toxicity.  
- Uses **toxic-bert** from Hugging Face.  
- Returns a predefined safe response if content is unsafe.  

### 2. RAGPipeline  
- Retrieves relevant documents using **FAISS** similarity search.  
- Supplies retrieved context to the LLM for grounded answers.  

### 3. HeuristicRouter  
- Chooses the appropriate LLM dynamically.  
- Routing logic considers:
  - Query length  
  - `bias_towards_35` metric (adjusted based on user feedback)  

### 4. ControlHintAdapter  
- Dynamically modifies the LLM prompt.  
- Default hint: *"Answer in a helpful, concise, and knowledge-sharing way"*  
- Updated with stronger factual grounding when user gives negative feedback.  

### 5. LLMService  
The **orchestrator** that ties everything together:  
- Applies Guardrail checks  
- Runs RAG retrieval  
- Routes to an LLM  
- Applies prompt adaptation  
- Logs metrics (latency, model selection) via **MLflow**  

---

## 🧪 Experimentation & HITL

The `chatbot_loop` function provides a CLI for interactive experimentation.  

- After each response, the user provides feedback:  
  - 👍 `y` → Positive feedback  
  - 👎 `n` → Negative feedback  
- Negative feedback triggers:  
  - **ControlHintAdapter update** → "Be more factual, detailed, and knowledge-grounded"  
  - **Router bias adjustment** → Favor **Claude-3.5-Sonnet**  

---

## 📦 MLflow Model Packaging & Registry

- **_PyfuncChatbot**: Wraps `LLMService` into an MLflow `pyfunc` model.  
- Input: Pandas DataFrame / Dict with `"question"` column.  
- **Artifacts logged** with the model:
  - `docs.json` → FAISS seed documents  
  - `adapter_hint.txt` → Initial adapter hint  

### CLI Commands
- `register_model` → Logs `_PyfuncChatbot` to MLflow Model Registry.  
- `promote_model` → Promotes the latest version to **Staging** / **Production**.  

---

## 🌐 Production Deployment

A **FastAPI** app provides the production-ready inference layer.

### Endpoints
- `GET /health` → Health check endpoint.  
- `POST /chat` → Accepts `ChatRequest` (user question) → Returns `ChatResponse` (answer).  

### Deployment
- **Dockerfile** → Containerize the chatbot.  
- **Kubernetes manifest (K8S_DEPLOYMENT)** → Scale & deploy on Kubernetes.  

---

## ⚙️ Tech Stack

- **LLMs**: OpenAI GPT, Anthropic Claude 3.5  
- **Safety**: Hugging Face `toxic-bert`  
- **Retrieval**: FAISS  
- **Experimentation**: MLflow (logging, registry, metrics)  
- **Serving**: FastAPI + Uvicorn  
- **Deployment**: Docker, Kubernetes  

---

## 🏁 Quickstart

```bash
# 1. Run chatbot loop for local testing
python chatbot.py

# 2. Register model in MLflow
python cli.py register_model --name llmops-chatbot

# 3. Promote to Production
python cli.py promote_model --name llmops-chatbot --stage Production

# 4. Launch FastAPI server
uvicorn server:app --host 0.0.0.0 --port 8000
