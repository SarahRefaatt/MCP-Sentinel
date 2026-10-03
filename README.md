# 🛡️ MCP Sentinel

### Secure. Approve. Audit. Automate.

**MCP Sentinel** is a security and permission-control framework for **Model Context Protocol (MCP)** tool execution. It provides a controlled layer between AI agents and MCP tools, allowing organizations to define permissions, assess operational risk, require human approval for sensitive actions, and maintain a complete audit trail.

The project demonstrates how MCP-based AI systems can move beyond simple tool calling toward **secure, policy-aware, and auditable agentic workflows**.

---

## ✨ Overview

AI agents can interact with files, systems, APIs, and other resources through MCP tools. While this provides powerful automation capabilities, unrestricted tool execution can introduce security risks.

MCP Sentinel addresses this by enforcing a configurable permission policy for every tool invocation.

Each operation can be configured with one of three policies:

- 🟢 **Allow** — Execute automatically
- 🔴 **Deny** — Block execution
- 🟡 **Ask** — Require explicit user approval

The system also assigns risk levels to operations and records execution decisions in an audit log.

```text
                 ┌─────────────────────┐
                 │      AI Agent       │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │    MCP Sentinel     │
                 │                     │
                 │  Permission Policy  │
                 │  Risk Assessment    │
                 │  Approval Workflow  │
                 │  Audit Logging      │
                 └──────────┬──────────┘
                            │
                  ┌─────────┼─────────┐
                  ▼         ▼         ▼
               ALLOW      ASK       DENY
                  │         │         │
                  ▼         ▼         ▼
               Execute   Approve    Block
```

---

## 🚀 Key Features

### 🔐 Permission Enforcement

Control MCP tools using three configurable policies:

| Policy | Behavior |
|---|---|
| **Allow** | Tool executes immediately |
| **Ask** | User approval is required |
| **Deny** | Tool execution is blocked |

### ⚠️ Risk Assessment

Tools are classified according to their potential impact:

- 🟢 Low
- 🔵 Medium
- 🟠 High
- 🔴 Critical

This allows the system to treat operations differently depending on their potential risk.

### 👤 Human-in-the-Loop

Sensitive operations can require explicit user approval before execution.

This prevents an AI agent from performing potentially destructive actions without human confirmation.

### 📋 Audit Logging

Every tool operation is recorded with information such as:

- Timestamp
- Tool name
- Arguments
- Permission policy
- Execution decision
- Approval status

Example:

```text
[2026-10-03 14:30:21] ALLOWED: read_file
Arguments: {'filepath': 'test.txt'}

[2026-10-03 14:31:04] DENIED: delete_file
Arguments: {'filepath': 'test.txt'}

[2026-10-03 14:32:18] ASK: write_file
Decision: Awaiting approval
```

### 🎯 Argument-Specific Permissions

Permissions can be applied with awareness of the operation's arguments, enabling more granular security controls.

### 🤖 AI-Powered Tool Execution

The AI Host provides a natural-language interface where users can request operations conversationally.

The system evaluates the requested operation before allowing the AI to execute the corresponding MCP tool.

### 📊 Security Visibility

The GUI exposes permission policies, risk information, and audit activity, making it easier to understand and monitor tool execution.

---

# 🏗️ Architecture

MCP Sentinel consists of four main components:

```text
┌───────────────────────────────────────────────────────┐
│                    MCP Sentinel                       │
│                                                       │
│  ┌───────────────┐        ┌───────────────────────┐  │
│  │   GUI Client  │        │      AI Host          │  │
│  │   Port 7863   │        │      Port 7864        │  │
│  └───────┬───────┘        └───────────┬───────────┘  │
│          │                            │              │
│          └────────────┬───────────────┘              │
│                       ▼                              │
│             ┌───────────────────┐                    │
│             │  Permission Base  │                    │
│             │      Client       │                    │
│             └─────────┬─────────┘                    │
│                       ▼                              │
│             ┌───────────────────┐                    │
│             │    MCP Server     │                    │
│             │   Risk-Tiered     │                    │
│             │      Tools        │                    │
│             └───────────────────┘                    │
└───────────────────────────────────────────────────────┘
```

---

# 📁 Project Components

| Component | Description |
|---|---|
| `mcp_permission_server.py` | MCP server containing risk-tiered tools |
| `mcp_permission_client_base.py` | Core permission enforcement and audit framework |
| `mcp_permission_client_app.py` | Gradio GUI for managing permissions and testing tools |
| `mcp_permission_host_app.py` | AI-powered host with risk-aware tool execution |
| `data/` | Sample files used for testing |

---

# ⚙️ Permission Model

MCP Sentinel supports three execution policies.

### Allow

Operations configured with `allow` execute automatically.

```text
User/AI Request
      ↓
Permission Check
      ↓
    ALLOW
      ↓
Tool Execution
      ↓
Audit Log
```

Example:

```text
read_file → ALLOW → Execute immediately
```

### Deny

Operations configured with `deny` are blocked.

```text
User/AI Request
      ↓
Permission Check
      ↓
    DENY
      ↓
Execution Blocked
      ↓
Audit Log
```

Example:

```text
delete_file → DENY → Permission denied
```

### Ask

Operations configured with `ask` require human approval.

```text
User/AI Request
      ↓
Permission Check
      ↓
     ASK
      ↓
Approval Request
      ↓
 ┌──────┴──────┐
 ▼             ▼
Approve       Reject
 │             │
 ▼             ▼
Execute       Block
```

Example:

```text
write_file → ASK → Human approval → Execute
```

---

# 🧪 Testing

## 1. Create Test Data

From the project directory:

```bash
cd mcp_security_lab
echo "Sample content for testing" > data/test.txt
```

---

# 🖥️ Test the GUI Client

Activate the environment:

```bash
cd mcp_security_lab
source ../mcp_security_env/bin/activate
```

Then start the GUI client:

```bash
python mcp_permission_client_app.py mcp_permission_server.py
```

The application should start on:

```text
http://127.0.0.1:7863
```

---

## Test 1 — Allow Policy

This test verifies that an allowed operation executes without user approval.

### Configure the permission

1. Open the **Permissions** tab.
2. Click **Load Tools**.
3. Select `read_file`.
4. Select **Allow**.
5. Click **Save Permission**.
6. Navigate to the **Tools** tab.

Call:

```text
Tool: read_file
Arguments: {"filepath": "test.txt"}
```

### Expected result

```text
Sample content for testing
```

The audit log should contain an entry similar to:

```text
[timestamp] ALLOWED: read_file
Arguments: {'filepath': 'test.txt'}
```

---

# Test 2 — Deny Policy

This test verifies that restricted operations are blocked automatically.

Configure:

```text
Tool: delete_file
Policy: Deny
```

Then attempt:

```text
Tool: delete_file
Arguments: {"filepath": "test.txt"}
```

### Expected result

```text
Permission denied for tool: delete_file
```

The audit log should contain:

```text
[timestamp] DENIED: delete_file
Arguments: {'filepath': 'test.txt'}
```

The file should remain unchanged.

---

# Test 3 — Ask Policy

This test demonstrates human-in-the-loop execution.

Configure:

```text
Tool: write_file
Policy: Ask
```

Then call:

```text
Tool: write_file
Arguments:
{
  "filepath": "new_file.txt",
  "content": "Test content"
}
```

The system should display a permission request similar to:

```text
Permission required for tool: write_file

This tool requires approval before execution.
Please approve this operation in the GUI to proceed.
```

Select **Approve & Execute**.

### Expected result

```text
Successfully wrote to new_file.txt
```

The audit log should record the approval flow:

```text
[timestamp] ASK: write_file
Decision: Awaiting approval

[timestamp] ALLOWED: write_file
Decision: Policy: ask
```

---

# 🤖 Test the AI Host

The AI Host provides a conversational interface for interacting with the MCP tools.

Start the application:

```bash
python mcp_permission_host_app.py mcp_permission_server.py
```

The application should be available at:

```text
http://127.0.0.1:7864
```

---

# Test 4 — AI Risk Assessment

Open the AI Host and review the **Permission Status** section.

### Low-Risk Operation

Ask:

```text
Read the contents of test.txt
```

Because `read_file` is configured as **Allow**, the operation should execute automatically.

Expected response:

```text
Sample content for testing
```

---

### High-Risk Operation

Ask:

```text
Delete the file test.txt
```

Because `delete_file` is configured as **Deny**, the operation should be blocked.

Expected behavior:

```text
Permission denied for tool: delete_file
```

The AI should not be able to bypass the configured policy.

---

### Medium-Risk Operation

Ask:

```text
Write a new file called greeting.txt with the content "Hello, world!"
```

Because `write_file` is configured as **Ask**, the system should request approval before executing the operation.

You should see a request similar to:

```text
Permission required for tool: write_file

Arguments:
{
  "filepath": "greeting.txt",
  "content": "Hello, world!"
}

This tool requires approval before execution.
```

Approve the operation.

Expected result:

```text
Operation approved and executed.

Successfully wrote to greeting.txt
```

The operation should also appear in the audit trail.

> **Note:** LLM tool-calling behavior can be non-deterministic. If the AI does not invoke the tool on the first attempt, retry the request or rephrase it.

---

# 🔍 Security Workflow

The complete execution flow is:

```text
Natural Language Request
          │
          ▼
      AI Host
          │
          ▼
    Tool Selection
          │
          ▼
    Risk Assessment
          │
          ▼
 Permission Evaluation
          │
     ┌────┼────┐
     ▼    ▼    ▼
   ALLOW ASK  DENY
     │    │    │
     │    ▼    │
     │ Approval│
     │    │    │
     └────┼────┘
          ▼
    Tool Execution
          │
          ▼
      Audit Log
```

This architecture creates a security boundary around MCP tool execution rather than allowing the AI model to directly control tools.

---

# 🛠️ Technology Stack

- **Python**
- **Model Context Protocol (MCP)**
- **Gradio**
- **LLM-based AI Host**
- **Permission Policies**
- **Risk Assessment**
- **Human-in-the-Loop Approval**
- **Audit Logging**

---

# 📌 Example Use Cases

MCP Sentinel can serve as a foundation for security controls in agentic AI systems such as:

- Enterprise AI assistants
- AI coding agents
- File-management agents
- Internal automation agents
- MCP-based enterprise applications
- Tool-calling LLM applications
- Human-in-the-loop workflows
- Security and compliance monitoring

For example, an enterprise AI assistant could automatically allow read-only operations while requiring approval for file modifications and blocking destructive operations.

---

# 🔮 Future Improvements

Potential extensions include:

- 🔑 Role-based access control (RBAC)
- 👥 User and team-specific permissions
- 🔐 Authentication and authorization
- 🧩 Tool-level and argument-level policies
- 📈 Advanced security dashboards
- 🚨 Real-time security alerts
- 📝 Persistent audit storage
- 🔄 Policy versioning
- 🧠 More sophisticated risk scoring
- ⏱️ Approval expiration and timeouts
- 🏢 Enterprise policy management
- 🔍 Anomaly detection for agent behavior
- 📊 Security analytics and reporting

---

# 🎯 What This Project Demonstrates

MCP Sentinel demonstrates several important concepts for secure agentic AI:

**Policy Enforcement**  
AI tool calls are evaluated against explicit security policies.

**Risk-Aware Execution**  
Different operations can receive different levels of security controls.

**Human Oversight**  
Sensitive operations can require explicit human approval.

**Auditability**  
Tool executions and permission decisions are recorded for traceability.

**AI + MCP Integration**  
An LLM can interact with MCP tools while remaining subject to security controls.

**Defense in Depth**  
Security decisions are enforced outside the model rather than relying solely on prompting the AI to behave safely.

---

# 📄 License

Add your preferred license here, such as MIT.

---

## 🛡️ MCP Sentinel

**A security layer for permission-aware and auditable AI tool execution.**

> **Secure the tools. Control the actions. Keep humans in the loop.**
