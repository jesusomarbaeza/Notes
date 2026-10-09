# Introduction to Model Context Protocol (MCP)
*Improved & Organized Notes*

---

## What Is the Model Context Protocol?

An open-source standard protocol for connecting AI applications to external tools and data.

* **Standard:** A shared agreement on how something should work so systems stay compatible.
* **Protocol:** A set of rules for communication that defines the format, order, and meaning of messages.
* **Open / Open-source:** Publicly available for anyone to use or build on (created by Anthropic).

> **Core Idea:** An LLM’s output is only as good as its context. MCP provides a standardized way to give AI applications access to real, up-to-date external information and capabilities beyond their training data.

---

## MCP Participants: Hosts, Clients, and Servers

| Role | Description | Examples |
| :--- | :--- | :--- |
| **Host** | AI application that interfaces with the LLM and may present UI to the user. Acts as container and orchestrator. | Claude Desktop, Cursor, VS Code, Amazon Q, custom apps |
| **Client** | Component running inside the host. Instantiated by the host as needed. Responsible for communicating with servers. | VS Code extensions, CLI tools |
| **Server** | Provides context (tools, resources, prompts). Can be local or remote. | Google Drive, Slack, GitHub, PostgreSQL, file system, Angular MCP server |

**Flow:** Host instantiates one or more clients $\rightarrow$ Clients connect to servers $\rightarrow$ Servers supply context $\rightarrow$ Host feeds that context to the LLM.

---

## MCP Primitives: Tools, Resources, and Prompts

Primitives are the smallest building blocks a server can provide.

* **Tool**
  * Executable function on the server that performs actions.
  * May or may not return data.
  * *Examples:* Query a database, call an API, perform a calculation, schedule a meeting, send a message.
* **Resource**
  * Contextual, read-only information for the AI.
  * *Examples:* File contents, database schemas, documentation, local project files.
* **Prompt**
  * Reusable template designed to be selected and used by a user.
  * Helps guide high-quality interactions.

These primitives are exchanged in a standardized way so any compliant client and server can work together.

---

## How Data Is Transferred

1. **Data Layer:** JSON-RPC (standardized JSON messages)
2. **Transport Layer:**
   * **Local:** Standard I/O (stdio)
   * **Remote:** HTTP / SSE

---

## Advantages of MCP

### The Integration Problem (Existing Alternatives & Their Limitations)

| Approach | Limitation |
| :--- | :--- |
| **Model-specific plugins** | Not portable across models/vendors |
| **Manual API integrations** | Does not scale; high maintenance |
| **Language-specific frameworks** | Tied to a specific runtime/language |

### Benefits of MCP

* **Open protocol:** Anyone can implement and contribute.
* **Pre-built integrations:** Ecosystem of ready-made servers.
* **Standardized structure:** Consistent mental model (tools, resources, prompts).
* **Client flexibility:** Works across different hosts and applications.
* **Language-agnostic:** Independent of programming language.
* **Reduced hallucinations:** Connects models to real, current data instead of relying solely on training data.

> **Key Insight:** Models are not trained on your data. Without good context, they fill gaps probabilistically and can be confidently wrong. MCP grounds the model in real information.

---

## MCP in Action

### Common Use Cases

* **Chatbots:** Answer questions with real data, look up information on demand, take actions on behalf of users.
* **IDE Assistants:** Read data outside the project, access up-to-date documentation and examples, run commands/tools.
* **AI Agents:** Access tools and data mid-task, adjust on the fly, coordinate multi-step workflows.

### Example Ecosystem

* **Example Hosts / Clients:** Claude Desktop, Cursor, Amazon Q, VS Code extensions, CLI tools, Custom applications.
* **Example Servers:** Google Drive, Slack, GitHub, File system, Databases (e.g., PostgreSQL), Framework-specific servers (e.g., Angular).

---

## The MCP Request-Response Flow

### Three Phases

1. **Initialization**
   * Establish connection
   * Exchange protocol version
   * Capability negotiation
2. **Operation**
   * Exchange messages
   * List and call tools
   * Retrieve resources
   * Use prompt templates
   * LLM receives results as context and generates output
3. **Shutdown**
   * Cleanly close the connection

### Capability Negotiation

| Side | Capabilities | Details |
| :--- | :--- | :--- |
| **Client** | Roots, Sampling | • **Roots:** File-system locations the client has permission to work in.<br>• **Sampling:** Allows server to request the client's LLM to process a prompt and return the result. |
| **Server** | Prompts, Resources, Tools | Exposes primitives for the client to discover and use. |

---

## The Lifecycle of Context

* An LLM’s output quality depends heavily on the quality and relevance of its context.
* **Sources of context via MCP:**
  * Tools (what actions are available and when to use them)
  * Resources (data)
  * Prompts (templates)
  * Tool results and previous interactions
  * User messages

> ⚠️ **Important Caution:** Managing context requires balance. Too much context can degrade performance (recency bias, "context rot") and even increase hallucinations. MCP provides the mechanism to supply good context, but developers must still curate and manage the context window carefully.

---

## Summary

MCP is an open standard that lets AI applications reliably connect to external tools and data. By standardizing the interaction between hosts, clients, and servers through primitives (tools, resources, prompts), it reduces integration friction, improves portability, and helps ground LLM outputs in real information—leading to more accurate and useful AI applications.