# CMPE295A Multi-Agent SRE Assistant

A professional-grade, multi-agent system designed to automate Site Reliability Engineering (SRE) tasks. This project leverages the **Model Context Protocol (MCP)** to abstract infrastructure tools, enabling a team of specialized AI agents to collaborate on complex troubleshooting, monitoring, and remediation workflows.

## 📖 Project Overview

The SRE Assistant is built on a **Supervisor-Worker** multi-agent architecture using **LangGraph**. It orchestrates specialized agents (Kubernetes, Logs, Metrics, Runbooks) to autonomously investigate incidents. The system is decoupled from specific backend implementations through an **MCP Integration Layer**, making it modular and extensible.

### Key Features
*   **Multi-Agent Collaboration**: A Supervisor agent plans investigations and delegates tasks to specialized worker agents.
*   **MCP Tool Abstraction**: Uses the Model Context Protocol to dynamically discover and utilize tools from backend services without hardcoded integrations.
*   **Mock Infrastructure**: Includes a complete, realistic mock backend (K8s, Logs, Metrics, Runbooks) for deterministic testing and development.
*   **Dual Interfaces**: Provides both a rich Web UI (HTML/JS) and a Streamlit dashboard for interaction.
*   **Extensible Design**: New agents and tools can be added via configuration and MCP servers without modifying the core agent logic.

---

## 🏗️ Architecture

The system consists of four main layers:

1.  **User Interface Layer**:
    *   **Web UI**: A modern, responsive interface (`ui/enhanced_ui_v2.html`) for chat and visualization.
    *   **Streamlit App**: A rapid prototyping dashboard (`ui/streamlit_app.py`).
2.  **Agent Core Layer (`sre_agent/`)**:
    *   **Runtime**: FastAPI-based server handling requests.
    *   **Orchestrator**: LangGraph state machine managing the Supervisor and Workers.
    *   **Agents**: Specialized nodes (Kubernetes, Logs, Metrics, Runbooks) configured via `agent_config.yaml`.
3.  **MCP Integration Layer (`mcp_servers/`)**:
    *   **MCP Client**: A Python client in the agent core that connects to multiple MCP servers.
    *   **MCP Servers**: Node.js-based servers (`@ivotoby/openapi-mcp-server`) that wrap backend OpenAPI specs and expose them as MCP tools.
4.  **Backend Services Layer (`backend/`)**:
    *   **Mock Servers**: Python FastAPI/HTTP servers simulating real infrastructure APIs.
    *   **Data**: Realistic JSON datasets for pods, logs, metrics, and runbooks.

---

## 📂 Repository Structure

```text
CMPE295A_Multi_Agent_SRE_Assistant/
├── backend/                  # Mock Infrastructure Services
│   ├── data/                 # JSON datasets (k8s, logs, metrics, runbooks)
│   ├── openapi_specs/        # OpenAPI 3.0 specifications for backend APIs
│   ├── servers/              # Python server implementations
│   └── scripts/              # Backend startup/shutdown scripts
├── mcp_servers/              # Model Context Protocol Layer
│   └── openapi/              # Dockerfile for OpenAPI-to-MCP bridge
├── sre_agent/                # Intelligent Agent Core
│   ├── config/               # Agent configuration (agent_config.yaml)
│   ├── agent_nodes.py        # Worker agent implementations
│   ├── agent_runtime.py      # FastAPI runtime for the agent
│   ├── graph_builder.py      # LangGraph state machine construction
│   ├── multi_agent_langgraph.py # Main orchestration logic
│   └── supervisor.py         # Supervisor agent logic
├── ui/                       # User Interfaces
│   ├── enhanced_ui_v2.html   # Production-grade Web UI
│   └── streamlit_app.py      # Streamlit dashboard
├── scripts/                  # Project-level utility scripts (K8s, migration)
├── docker-compose.yml        # (If present) Container orchestration
├── pyproject.toml            # Python dependencies and project metadata
└── Makefile                  # Development tasks (lint, test, build)
```

---

## 🚀 Getting Started

### Prerequisites
*   **Python 3.12+**
*   **Node.js & npm** (for building MCP servers)
*   **Docker** (optional, for containerized run)
*   **LLM API Key**: Groq or Anthropic API key.

### 1. Environment Setup
Create a `.env` file in the `sre_agent` directory (or root, depending on execution context):

```bash
cp sre_agent/.env.example sre_agent/.env
```

Edit `.env` to add your API keys:
```ini
LLM_PROVIDER=groq  # or anthropic
GROQ_API_KEY=your_key_here
ANTHROPIC_API_KEY=your_key_here

# MCP Server URIs (Default for local execution)
MCP_K8S_URI=http://localhost:3000/mcp
MCP_LOGS_URI=http://localhost:3001/mcp
MCP_METRICS_URI=http://localhost:3002/mcp
MCP_RUNBOOKS_URI=http://localhost:3003/mcp
```

### 2. Start the Backend
The backend simulates the infrastructure the agents will investigate.

```bash
cd backend
./scripts/start_demo_backend.sh
```
*This starts 4 mock servers on ports 8001-8004.*

### 3. Start MCP Servers
(Note: For local development without Docker, you may need to run the MCP servers manually or ensure the Agent connects directly if using a simplified setup. The standard path uses the Docker image defined in `mcp_servers/openapi`).

If running locally without Docker, ensure you have an MCP-compliant server running that points to the backend ports (8001-8004).

### 4. Run the Agent
Install dependencies and start the agent runtime.

```bash
# Install dependencies
pip install -e .

# Run the agent CLI (Interactive Mode)
python -m sre_agent.cli
```

Or run the API server for the UI:
```bash
uvicorn sre_agent.agent_runtime:app --reload --port 8000
```

### 5. Launch the UI
Open `ui/enhanced_ui_v2.html` in your browser. It is configured to connect to `http://localhost:8000`.

---

## 🧩 Configuration

### Agent Configuration
Agents are defined in `sre_agent/config/agent_config.yaml`. This file maps specific agents to the tools they are allowed to use.

```yaml
agents:
  kubernetes_agent:
    name: "Kubernetes Infrastructure Agent"
    tools: [get_pod_status, get_deployment_status, ...]
```

### Backend Data
The mock data is located in `backend/data/`. You can modify the JSON files in `k8s_data`, `logs_data`, etc., to simulate different incident scenarios (e.g., crash loops, high latency).

---

## 🛠️ Development

### Makefile Commands
Use the `Makefile` for common tasks:
*   `make format`: Format code with Black and Isort.
*   `make lint`: Run flake8 and mypy.
*   `make test`: Run unit tests.

### Adding a New Tool
1.  **Backend**: Add the endpoint to the appropriate server in `backend/servers/`.
2.  **Spec**: Update the OpenAPI spec in `backend/openapi_specs/`.
3.  **MCP**: Rebuild/restart the MCP server to pick up the new spec.
4.  **Agent**: Add the new tool name to the appropriate agent in `sre_agent/config/agent_config.yaml`.

---

## 🔍 Troubleshooting

*   **Agent fails to connect**: Check `MCP_*_URI` variables in `.env`. Ensure backend servers are running.
*   **Tools not found**: Verify the MCP server has successfully loaded the OpenAPI spec. Check agent logs for "Discovered X tools".
*   **LLM Errors**: Verify `GROQ_API_KEY` or `ANTHROPIC_API_KEY` is set and valid.

---

**Author**: CMPE295A Team
**License**: MIT
