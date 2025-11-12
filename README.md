# Multi-Agent System for Research Analysis and Generation Automation

## Overview
This project automates AI-driven research report generation using a multi-agent LLM architecture integrated with Azure-based CI/CD pipelines. It enables automated literature search, analysis, and structured report creation, supported by a FastAPI backend, Docker containerization, and Jenkins automation.

---

## Tech Stack
| Component | Technology |
|------------|-------------|
| Backend | FastAPI |
| Agents | LangChain / CrewAI |
| Search Integration | Tavily API |
| Database | SQLite |
| Containerization | Docker |
| CI/CD | Jenkins, GitHub Actions |
| Cloud Deployment | Microsoft Azure (VM, ACR, CLI) |
| Monitoring | Jenkins Logs, Azure Dashboard |

---

## System Architecture
The system consists of multiple specialized AI agents coordinated by an Orchestrator:

1. Search Agent → Retrieves research papers from APIs (Tavily, Arxiv).
2. Reader Agent → Extracts and summarizes content.
3. Analyst Agent → Analyzes findings and patterns.
4. Generator Agent → Produces structured research reports.
5. Coordinator (Orchestrator) → Manages communication among agents and ensures workflow completion.

These agents interact asynchronously, exchanging results through shared memory and prompt engineering.

---

## Project Setup (Local Environment)

### 1. Install `uv` Environment Manager
```bash
uv --version
pip install uv
uv init <project_name>
uv venv
.venv\Scripts\activate.bat
uv pip install -r requirements.txt
uv add ipykernel
```

### 2. API Key Configuration
Get a Tavily Search API key for research retrieval:  
https://app.tavily.com/home  

Create a `.env` file and add:
```
TAVILY_API_KEY=your_key
OPENAI_API_KEY=your_key
GROQ_API_KEY=your_key
```

### 3. Run FastAPI Application
```bash
uvicorn research_and_analyst.api.main:app --reload
# or for public access
uvicorn research_and_analyst.api.main:app --host 0.0.0.0 --port 8000
```

### 4. View Database
Install the SQLite Viewer extension in VS Code to inspect `user.db`.

---

## Azure Cloud Deployment

### 1. Install Azure CLI
https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows  
Verify installation:
```bash
az --version
```

### 2. Authenticate and Configure Subscription
```bash
az login --use-device-code
az account set --subscription <YOUR_SUBSCRIPTION_ID>
```

If issues occur:
```bash
az logout
az account clear
rmdir /s /q "%USERPROFILE%\.azure"
az login --use-device-code
```

---

## Jenkins CI/CD Pipeline (Self-Hosted on Azure)

### Stage 1: Jenkins Setup
Run:
```bash
bash azure-deploy-jenkins.sh
```
This deploys Jenkins inside a Docker container on Azure and provides:
- Jenkins URL  
- Initial Admin Password  

### Stage 2: Infrastructure Setup
```bash
bash setup-app-infrastructure.sh
```
This sets up the Azure resources and generates a connection secret between Jenkins and Azure.

### Stage 3: Service Principal Creation
```bash
az ad sp create-for-rbac \
  --name "jenkins-research-report-sp" \
  --role Contributor \
  --scopes /subscriptions/$(az account show --query id -o tsv)
```

Configure the following in Jenkins global environment variables:
```
azure-client-id
azure-tenant-id
azure-client-secret
azure-subscription-id
```

---

## Docker Image Build and Push
Once Jenkins is running:
```bash
bash build-and-push-docker-image.sh
```
This:
- Builds the Docker image
- Pushes it to Azure Container Registry (ACR)

---

## Jenkins Pipeline Configuration
1. Create a new pipeline in Jenkins:
   - New Item → Pipeline
2. Add GitHub repository webhook for automatic triggers on push.
3. Configure the pipeline script to:
   - Pull code
   - Build Docker image
   - Push to ACR
   - Deploy container to Azure

---

## Output
- Automated retrieval and analysis of research papers.
- AI-generated structured research reports.
- FastAPI dashboard for:
  - Viewing agent logs
  - Monitoring tasks
  - Visualizing report summaries

---

## Directory Structure
```
research_and_analyst/
├── api/
│   ├── main.py
│   ├── routes/
│   └── utils/
├── agents/
│   ├── search_agent.py
│   ├── reader_agent.py
│   ├── analyst_agent.py
│   └── generator_agent.py
├── database/
│   └── user.db
├── Dockerfile
├── requirements.txt
├── setup-app-infrastructure.sh
└── azure-deploy-jenkins.sh
```

---

## Environment Variables
| Key | Description |
|-----|-------------|
| OPENAI_API_KEY | OpenAI access for text generation |
| TAVILY_API_KEY | For research and search automation |
| GROQ_API_KEY | For advanced reasoning models |
| LLM_PROVIDER | Selects model provider (OpenAI / Groq) |

---

## Monitoring and Logs
- Jenkins build logs for deployment cycles.
- Azure portal for container and VM monitoring.
- FastAPI logs for agent interactions and workflow tracking.

---

## Final Outcome
A production-ready, multi-agent AI research automation system that:
- Searches, reads, analyzes, and generates reports automatically.
- Is fully deployed via Dockerized Jenkins CI/CD on Azure Cloud.
- Offers real-time monitoring, secure deployment, and scalable automation.
