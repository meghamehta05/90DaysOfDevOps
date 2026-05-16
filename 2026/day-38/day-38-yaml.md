person.yaml
name: Megha Mehta
role: DevOps Learner
experience_years: 0
learning: true

tools:
  - Docker
  - Git
  - GitHub
  - Linux
  - Kubernetes

hobbies: [drawing, exploring, coding, listening_music, travel]
server.yaml
server:
  name: dev-server
  ip: 192.168.1.10
  port: 8080

database:
  host: localhost
  name: app_db
  credentials:
    user: admin
    password: secret123

startup_script: |
  #!/bin/bash
  echo "Starting server..."
  npm install
  npm run dev

startup_script_folded: >
  #!/bin/bash
  echo "Starting server..."
  npm install
  npm run dev

 day-38-yaml.md
# Day 38 – YAML Basics

## What I Learned

### 1. YAML Syntax Rules
- YAML uses indentation (NOT tabs)
- 2 spaces is standard practice
- Structure defines meaning

### 2. Lists in YAML
Two ways to write lists:
- Block style:
  ```yaml
  tools:
    - Docker
    - Git

Inline style:

tools: [Docker, Git]
3. Nested Objects
YAML supports deep nesting using indentation
Useful for structured configs like servers and databases
Key Difference: | vs >
| → preserves line breaks exactly (good for scripts)
> → folds lines into a single paragraph (good for logs/text)
Common Errors
Using tabs instead of spaces breaks YAML
Wrong indentation changes structure completely
Lists must align properly
Validation Learning
yamllint helps catch indentation errors
Even a single space mistake can break parsing
Final Takeaway

YAML is simple but extremely sensitive — indentation is everything.


---

If you want, I can next:
✔ :contentReference[oaicite:0]{index=0}  
✔ or :contentReference[oaicite:1]{index=1}