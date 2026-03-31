# Brickform

**Declarative Infrastructure for Agents. Agent as Code (AaC).**

Brickform is a conceptual open-source orchestration layer that aims to allow you to define, deploy, and manage AI Agents across any infrastructure using a declarative schema (JSON/YAML). It explores solutions to the "Agent Drift" and governance problems by turning shaky Python scripts into reliable, version-controlled infrastructure.

> **Note:** This project is currently in the **Idea / RFC (Request for Comments) phase**. The concepts, schemas, and architecture detailed below are highly experimental and bound to evolve as the community refines them.

## 🚀 The Problem

The industry is currently facing an **"Agent Sprawl" crisis**. Companies have dozens of hardcoded "custom GPTs," LangChain scripts, and LangGraph DAGs running everywhere. When moving from GPT-4o to Claude 3.5 or Ollama, or when changing a tool definition, everything breaks. 
Current frameworks are **Imperative** you write Python to make things happen, leading to model lock-in and a governance nightmare.

## 💡 The Solution: Agent as Code (AaC)

Brickform introduces a **Declarative** approach. Instead of hardcoding your agent logic in Python scripts, you describe *what* the agent is in a simple schema file. Brickform then deploys that Agent definition to any infrastructure (Docker, Kubernetes, Lambda, etc.). When the Agent runs, it connects to external services via API keys:
- **LLM providers** (OpenAI, Anthropic, Ollama, etc.)
- **Tools and MCP servers** (Google Search, Slack, file systems, etc.)
- **Memory backends** (DynamoDB, Redis, Vector DBs, etc.)

### Key Innovations

1. **Decoupled Logic and Memory**: 
   - **Logic (The "Agent")**: The orchestration, reasoning, planning, and tool-calling layer defined in your schema.
   - **Memory (The "State")**: The short-term session state (in DynamoDB or Redis) and long-term knowledge (Vector DBs).
   - By decoupling these, you can redeploy the Agent logic without losing its memory, or swap memory backends without redeploying.

2. **Model-Agnostic**: 
   - Point your Agent at any LLM provider (OpenAI, Anthropic, Ollama, etc.) via API keys in your schema or configuration.
   - Change the LLM provider in your schema without rewriting any code.

3. **Multi-Agent Orchestration**:
   - Define entire systems of agents as a graph (DAG) in your schema.
   - Agents can interact asynchronously by publishing and subscribing to shared topics, allowing for complex, collaborative workflows without passing massive context windows.

4. **Portable & Reusable Agent Logic**:
   - The schema can reference pre-built and versioned agent logic packages from a registry (similar to Docker images).
   - This allows teams to share and reuse standardized agents (e.g., a `company/slack_notifier:1.2.0`) or deploy custom-built logic, promoting consistency and reducing redundant code.

## 🛠️ The Envisioned Architecture (MVP)

The following represents the proposed architecture we aim to build:

1. **The Specification (Universal Agent Schema)**
   - Define a standard schema representing an `agent` resource, including its role, memory backend (e.g., `dynamodb`), and tools (e.g., `mcp: google_search`).

2. **The Proposed CLI Engine (`brickform`)**
   - **Validate**: Checks if the JSON/YAML schema is syntactically and semantically correct.
   - **Plan**: Shows the user what agents will be deployed and where ("Deploying research_agent to AWS Lambda, writer_agent to local Docker").
   - **Apply**: Provisions the Agent infrastructure (containers, serverless functions, etc.) and connects them to their configured LLM providers, tools, and state backends.

3. **The Agent Runtime**
   - A lightweight, portable runtime that runs your Agent logic, pulls configuration from the schema, connects to its configured LLM provider via API keys, calls tools and MCP servers, and manages state in its memory backend.

## 📄 Example Schema (Concept)

```yaml
# A multi-agent system definition for research and reporting
kind: "AgentSystem"
name: "research_and_report"

agents:
  - resource: "agent"
    name: "research_agent"
    # The agent's logic could be a pre-built package
    # from: "brickform-registry/universal_researcher:1.0"
    role: "Research assistant that gathers information on topics."
    system_prompt: "You are a helpful research assistant. Find information and publish a summary."
    llm:
      provider: "openai"
      model: "gpt-4o"
      api_key_env: "OPENAI_API_KEY"
    tools:
      - type: "mcp"
        name: "google_search"
        api_key_env: "GOOGLE_SEARCH_API_KEY"
    memory:
      type: "dynamodb"
      table: "agent_sessions"
      region: "us-east-1"
    # Defines a topic it can publish its findings to
    outputs:
      - topic: "research_summary"

  - resource: "agent"
    name: "writer_agent"
    role: "Writes polished reports based on research summaries and posts to Slack."
    llm:
      provider: "anthropic"
      model: "claude-3.5-sonnet"
      api_key_env: "ANTHROPIC_API_KEY"
    # Subscribes to the topic from the research agent
    inputs:
      - topic: "research_summary"
    tools:
      - type: "mcp"
        name: "slack"
        api_key_env: "SLACK_BOT_TOKEN"
```
