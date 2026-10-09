# Model Context Protocol (MCP) in Practice
## Python-Focused Reference Notes

> Based on the training **"Orchestrating AI Agents with MCP and Oracle AI Database"**. These notes adapt the training into a practical Python reference.

## 1. MCP in one sentence

The Model Context Protocol (MCP) is a standard interface that lets an AI application discover and use external **tools**, **resources**, and **prompts** without requiring a custom integration for every system.

A useful mental model:
```text
User -> AI host/client -> LLM decides what is needed -> MCP server -> database/API/files
```
The LLM plans the work. The MCP server performs only the operations it explicitly exposes.

## 2. Core MCP concepts

### Tools

Callable operations with typed inputs and outputs. In Python, a tool is commonly a type-hinted function decorated with `@mcp.tool()`.

Examples:

- Run an approved database query
- Retrieve a complaint record
- Validate test data
- Create a test-evidence artifact
- Call an internal REST API

### Resources

Read-only or file-like context that clients can retrieve.

Examples:

- Schema documentation
- API specifications
- Reference data
- Test standards
- Application configuration that is safe to expose

### Prompts

Reusable prompt templates supplied by the server.

Examples:

- Analyze a failed API test
- Generate a BDD scenario from requirements
- Review database evidence

## 3. Local versus remote MCP servers

### Local server

- Runs on the developer's workstation
- Commonly communicates over **STDIO**
- Good for development, database exploration, and troubleshooting
- Frequently controlled directly in the terminal
- Credentials remain under the local user's control

### Remote server

- Runs as a shared service
- Commonly uses **Streamable HTTP/HTTPS**
- Better for centralized governance and enterprise access
- Usually requires authentication, authorization, logging, and operational support

## 4. Python MCP quick start

### Create the project with `uv`

```powershell
uv init python-mcp-demo
cd python-mcp-demo
uv add "mcp[cli]"
```

The current official Python SDK requires Python 3.10 or newer.

### Minimal server

Create `server.py`:

```python
from mcp.server import FastMCP

mcp = FastMCP("Python MCP Demo")

@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two integers."""
    return a + b

@mcp.resource("app://about")
def about() -> str:
    """Return basic information about this MCP server."""
    return "Python MCP demo with one tool and one resource."
```

Python type hints define the tool's input schema. The docstring helps the AI understand when the tool should be used.

### Run with MCP Inspector

```powershell
uv run mcp dev server.py
```

Use Inspector to confirm:
1. The server starts.
2. The `add` tool is listed.
3. Inputs are validated.
4. Tool call returns the expected result.
5. The resource can be read.

### Run over HTTP

```powershell
uv run mcp run server.py --transport streamable-http
```

Use HTTP for remote or service-style deployments. Use STDIO for local developer integrations.

## 5. Testing a Python MCP server

The official SDK supports an in-memory client, so tests do not need a subprocess, open port, or external host.

Create `test_server.py`:

```python
import pytest
from mcp import Client
from server import mcp

@pytest.mark.anyio
async def test_add():
    async with Client(mcp) as client:
        result = await client.call_tool("add", {"a": 2, "b": 3})
        assert result.structured_content == {"result": 5}
```

Install and run the test tools:

```powershell
uv add --dev pytest anyio
uv run pytest -v
```

Test at least these cases for every tool:

- Expected valid input
- Boundary values
- Missing or invalid input
- Authorization failure
- Latency timeout or unavailable service
- Empty result
- Sensitive data redaction
- Audit information, when required

## 6. Python tool design pattern

Keep MCP registration thin. Put business logic in normal Python services so it can be tested without MCP.

```text
src/
  app/
    __init__.py
    server.py
    services/
      complaint_service.py
    repositories/
      complaint_repository.py
    models/
      complaint.py
  tests/
    test_server.py
    test_complaint_service.py
```

Example:

```python
from dataclasses import asdict

from mcp.server import FastMCP
from app.services.complaint_service import ComplaintService

mcp = FastMCP("Complaint Tools")
service = ComplaintService()

@mcp.tool()
def get_complaint(complaint_id: str) -> dict:
    """Retrieve an approved, non-sensitive complaint view by ID."""
    complaint = service.get_by_id(complaint_id)
    return asdict(complaint)
```

Recommended separation:

- **`server.py`**: MCP decorators and transport setup
- **`services/`**: Business rules and validation
- **`repositories/`**: Oracle/API file access
- **`models/`**: Typed request and response objects
- **`tests/`**: Unit, integration, and authorization tests

## 7. Oracle database patterns from the training

The training describes three Oracle-oriented MCP choices:

### SQLcl MCP

- Local, stateful, STDIO-based server
- Intended for developers, DBAs, and power users
- Provides operations such as listing connections, connecting, running SQL or SQLcl commands, retrieving schema information, and disconnecting
- Keeps credentials out of the LLM and stores passwords or certificates in Oracle wallets

### ORDS MCP

- Remote, Streamable HTTPS server
- Supports OAuth2/JWT-based access
- Can expose multiple Oracle database pools
- Fits centralized API gateways, web apps, and enterprise AI tools
- Propagates identity and writes MCP activity to database audit logs

### OCI Managed MCP

- Oracle-managed option for remote MCP services
- Integrated with OCI IAM security
- Automates connection discovery, SQL execution, and schema-information tools
- Propagates identity and writes MCP activity to database audit logs

## 8. Calling Oracle from Python safely

For your own Python MCP server, keep direct database access behind a repository. This is a generic pattern, not a replacement for your organization's approved connection method.

```python
import oracledb

class ComplaintRepository:
    def __init__(self, dsn: str, user: str, password: str) -> None:
        self.dsn = dsn
        self.user = user
        self.password = password

    def find_status(self, complaint_id: str) -> str | None:
        sql = """
            SELECT status
            FROM complaints
            WHERE complaint_id = :complaint_id
        """
        with oracledb.connect(
            user=self.user,
            password=self.password,
            dsn=self.dsn,
        ) as connection:
            with connection.cursor() as cursor:
                cursor.execute(sql, complaint_id=complaint_id)
                row = cursor.fetchone()
                return str(row[0]) if row else None
```

Important practices:

- Always use least-privilege database accounts.
- Always use bind variables.
- Never store passwords, credentials, or connection strings in source control.
- Always log tool calls with a unique correlation ID.
- Never output full database errors or tracebacks to the client without logging them securely first.
- Validate every tool argument before database use.
- Return the minimum fields needed.
- Log tool name, caller identity, result status, and correlation ID without logging secrets.

## 9. Security checklist

The training emphasizes least privilege, segregated environments, explicit review, and avoiding automatic approval of tool actions.

Before exposing a Python function as an MCP tool, verify:

- [ ] The operation is necessary.
- [ ] Inputs are typed and validated.
- [ ] Read and write actions are segregated.
- [ ] Authorization is checked for the specific operation and data.
- [ ] The database account has only required privileges.
- [ ] Development and test databases use sanitized data.
- [ ] High-risk operations require stronger controls and monitoring.
- [ ] Destructive actions require explicit confirmation.
- [ ] Secrets are never returned to the LLM.
- [ ] Tool requests and results are auditable.
- [ ] Errors do not reveal SQL, credentials, or internal topology.
- [ ] Rate limits and timeouts are configured.

## 10. STDIO logging rule

Do not use `print()` for logging in a STDIO MCP server because standard output carries protocol messages. Use Python logging, which normally writes to standard error.

```python
import logging

logger = logging.getLogger(__name__)

def process_request(request_id: str) -> None:
    logger.info("Processing request %s", request_id)
```

## 11. Practical applications for Python automation

Keep generation and review logic in Python services. MCP should be the controlled interface, not the entire application architecture.

Do not expose unrestricted `run_sql` when smaller purpose-built tools can do the job.

### Existing FastAPI app

The official SDK documentation supports running MCP over HTTP and integrating it with an existing FastAPI application. Treat the MCP endpoint as another protected application surface with the same authentication, authorization, observability, and deployment controls as the rest of the API.

## 12. Tool design: broad versus curated

Avoid a tool like:

```text
run_any_sql()
```

Prefer tools like:

```text
get_complaint_status(complaint_id)
find_test_records(criteria)
assert_test_result(complaint_id, expected_values)
```

Curated tools are easier to secure, validate, test, audit, and explain to reviewers.

## 13. Troubleshooting guide

### Server does not appear

- Run `uv run mcp dev server.py`.
- Confirm the decorated `mcp` object is at module scope.
- Confirm the installed SDK version matches the documentation you are using.

### Tool is listed but fails

- Check the Python type hints and argument names.
- Confirm the tool is using the intended environment and connection pool.
- Add a unit test for the failing input.
- Test the service function directly.

### STDIO connection breaks

- Remove `print()` calls.
- Send logs to standard error through `logging`.
- Confirm no imported module writes banners or debug output to standard output.

### Database authorization fails

- Verify the database user or JWT has the required role or scope.
- Confirm the tool is using the intended environment and connection pool.
- Verify authorization independently of the LLM.
- Preserve the correlation ID for investigation.

### Results are too large

- Return only necessary columns.
- Add pagination parameters.
- Return summaries plus references rather than entire datasets.
- Set maximum result limits in the server, not only in the prompt.

## 14. Suggested learning exercise

Build a safe, local Python MCP server with three capabilities:

1. **Tool**: validate a complaint ID format.
2. **Resource**: expose a short testing-standard document.
3. **Prompt**: create a template for summarizing test evidence.

Then:

1. Run it with MCP Inspector.
2. Add in-memory tests.
3. Add structured logging.
4. Introduce a mocked repository.
5. Test invalid input and dependency failure.
6. Only after that, connect to an approved SIT dependency.

## 15. Key takeaways

- MCP standardizes how AI applications use external capabilities.
- Servers expose tools, resources, and prompts.
- Python type hints and docstrings help define clear tool contracts.
- Keep business logic outside the MCP registration layer.
- Prefer narrow, curated tools over unrestricted database or shell access.
- Use least privilege, sanitized non-production data, explicit approval, and full auditing.
- Test tools through an in-memory client before testing transports or real dependencies.
- Treat an MCP server as a real application boundary, not merely a prompt extension.

## References

- Training source: "Orchestrating AI Agents with MCP and Oracle AI Database".
- Official MCP Python SDK: https://github.com/modelcontextprotocol/python-sdk
- Current Python SDK documentation: https://py.sdk.modelcontextprotocol.io/
- Model Context Protocol documentation: https://modelcontextprotocol.io/

> Version note: MCP SDKs evolve. Check the current official Python SDK documentation when starting or upgrading a project.