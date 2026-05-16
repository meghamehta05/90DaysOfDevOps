 Day 87 -- Introduction to Agentic AI for DevOps

## Overview
Today I explored Agentic AI and built my first AI-powered DevOps tools using Ollama, LangChain, and Docker CLI integrations.

---

## What is Agentic AI?

An AI agent is an LLM that can:
- Think (reason about a problem)
- Act (use tools like CLI commands or APIs)
- Observe (read tool outputs and decide next steps)

Unlike a chatbot, it does not just reply — it performs actions in the system.

---

## Why Agentic AI for DevOps?

DevOps involves many CLI tools:
- docker
- kubectl
- terraform
- aws cli

Agentic AI allows:
- Automatic debugging
- Infrastructure diagnosis
- Self-healing workflows
- Faster root cause analysis

---

## ReAct Pattern (Reason + Act + Observe)

Example:

User: "Why is broken-app crashing?"

1. Reason: Check running containers
2. Act: docker ps -a
3. Observe: container is restarting

4. Reason: Check logs
5. Act: docker logs broken-app
6. Observe: exit code 1

7. Reason: Container exits immediately after start
8. Final Answer: App crashes due to failing startup command

---

## Setup Summary

Tools installed:
- Ollama (local LLM runtime)
- Gemma 4 model
- Python virtual environment
- LangChain + LangGraph
- Docker CLI integration

Verification:
✔ Python 3.10+  
✔ Docker running  
✔ kubectl available  
✔ Ollama + gemma4 installed  

---

## Docker Error Explainer (Module 1)

### What it does
- Takes raw Docker error messages
- Uses LLM (Gemma 4)
- Returns:
  - Root cause
  - Explanation
  - Fix commands

### Example
Error:

port is already allocated


Output:
- Another service is using the port
- Kill or change port mapping
- Fix: `docker run -p 8081:8080`

---

## Docker Troubleshooter Agent (Module 2)

### Tools used by agent:
- list_containers()
- get_logs()
- inspect_container()

### What happened:
- Agent automatically detected container state
- Read logs without being told
- Identified crash reason
- Explained root cause

---

## ReAct Architecture


User Input
↓
LLM (Gemma 4)
↓
Reason → Act → Observe loop
↓
Tools (Docker CLI)
↓
Final Answer


---

## Tool I Added

### list_images tool

Purpose:
- Lists Docker images and sizes

Effect:
- Agent can now analyze disk usage and image inventory

---

## System Prompt Impact

System prompt defines:
- Role of model (Docker expert)
- Output format
- Level of detail

Lower temperature (0.3):
- More stable responses
- Better for debugging tasks



## Key Learnings

- AI agents can execute real system commands
- ReAct loop enables reasoning + execution
- Tools extend LLM capabilities beyond text
- DevOps automation can be fully AI-driven
- CLI tools become “AI actions”



## Final Insight

Agentic AI turns DevOps into:

FROM:
Manual debugging + CLI commands

TO:
Autonomous AI diagnosing infrastructure using tools