Day 88 is where your “agent” stops being a demo and starts behaving like a real DevOps assistant that can reason across systems instead of just calling one script.

Here’s a clean, structured way to complete it and also understand what actually matters.

 Core Idea (What you are building today)

You are extending this:

LLM (Gemma)
  ↓
Tools (Docker only)

Into this:

LLM (Gemma)
  ↓
Tool Router (ReAct reasoning)
  ↓
Docker Tools + Kubernetes Tools + CI/CD Tools
  ↓
MCP Server (shared tool layer)
  ↓
Any AI client can use same tools

This is the jump from “AI script” → “AI system”.

 Task 1: Multi-Tool Agent (Docker + Kubernetes)
 What you should verify

Your agent should have 6 tools total:

Docker tools
list_containers
get_logs
inspect_container
Kubernetes tools
list_pods
describe_pod
get_events
 Test broken systems
1. Broken Pod
kubectl apply -f module-3/broken_pod.yaml
kubectl get pods

You should see:

CrashLoopBackOff
2. Broken Docker container
docker run -d --name broken-container nginx:alpine sh -c "echo start && sleep 2 && exit 1"
 What the agent should do
Example query:
Why is broken-pod crashing?

Expected internal flow:

Reason: need pod status
→ list_pods()

Reason: pod is failing
→ describe_pod()

Reason: check logs/events
→ get_events()
Example cross-domain query:
What is broken in my system?

Expected:

Docker tools + Kubernetes tools both used
Correlation in final answer
 Key Learning

You are not coding logic.

You are designing tool boundaries.

LLM decides:

which tool
when
how many times
🔌 Task 2: MCP (Most important concept today)
What MCP actually is (simple)

MCP =

“USB port for AI tools”

Instead of hardcoding tools inside Python:

 Old way
tools = [docker_tool, k8s_tool]
 MCP way
Tools live in a server
Any AI can plug in and use them
Architecture
        MCP SERVER
     (kubectl tools)
           ↑
           ↓
   MCP CLIENT (LangChain / Claude / Cursor)
           ↓
        LLM (Gemma)
Why DevOps cares

Without MCP:

tools are duplicated in every agent
tightly coupled code

With MCP:

one tool server
multiple AI clients
reusable DevOps “tool APIs”
Real DevOps analogy
MCP Concept	DevOps Equivalent
MCP Server	Kubernetes API
Tools	kubectl commands
Client	DevOps engineer / CI system
 Task 3: MCP Server (what matters)

Your server:

@mcp.tool
def list_pods():
Key idea:

This is now a tool API service

Not Python logic.

What changes with MCP
Before

Agent owns tools

After

Agent DISCOVERS tools

Agent → asks MCP → gets tool list → uses dynamically
Why this is powerful

Tomorrow:

VS Code can use your kubectl tools
Claude Desktop can debug your cluster
Your Python agent can reuse same tools

No rewriting.

 Task 4: CI/CD Analyzer (real DevOps use case)

This is the most practical part of today.

Tools you built:
1. List failed workflows
gh run list --status failure
2. Fetch logs
gh run view --log-failed
3. Read workflow file
.github/workflows/*.yml
 Agent behavior

User:

Why did CI fail?

Agent flow:

→ list_workflow_runs()
→ get_failed_logs()
→ get_workflow_file()
→ LLM summarizes root cause
Real-world mapping
Tool	Real DevOps Use
gh run list	CI monitoring
gh logs	debugging pipeline
workflow file	root cause analysis
🛠 Task 5: Your custom tool (important mindset shift)

You are now doing:

“turning CLI into intelligence layer”

Examples:

Kubernetes search tool
kubectl logs + grep
Terraform analyzer
terraform plan
AWS checker
aws ec2 describe-instances
Key rule

If CLI exists → it can become an agent tool.

 Mental Model of Today

You built 3 layers:

1. Tool Layer
kubectl, docker, gh
2. Intelligence Layer
LLM decides:
- what to run
- when to run it
- how to interpret results
3. Protocol Layer (MCP)
Tool sharing system across AI apps
⚠️ Common mistakes
 Too many tools

→ agent gets confused

 Bad docstrings

→ wrong tool selection

 Huge logs

→ LLM overload

Fix = keep tools:

small
clear
single purpose
 What you should write in your markdown

Include:

✔ Architecture diagram
User → LLM → Tool Router → Docker/K8s/CI → Output
✔ MCP explanation
server
client
protocol advantage
✔ CI/CD analyzer flow
✔ One real failure example
Final takeaway

Today you did something important:

You moved from “writing scripts” → “designing autonomous troubleshooting systems”

And the real DevOps insight is:

Modern DevOps = CLI + AI reasoning layer