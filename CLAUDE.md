# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Uses `uv` for environment and dependency management (Python >=3.10).

```bash
uv venv && uv pip install -e .        # setup (on Windows activate with .venv\Scripts\activate)
uv run main.py                        # start the MCP server (stdio; runs until stopped)
uv run pytest                         # run all tests
uv run pytest tests/test_document.py::TestBinaryDocumentToMarkdown::test_binary_document_to_markdown_with_pdf  # single test
uv run mcp dev main.py                # MCP Inspector for interactive tool testing (mcp[cli] is installed)
```

No linter or formatter is configured.

## Conventions

- Always apply appropriate type hints to function arguments (and return values).

## Architecture

An MCP server (`FastMCP` from the `mcp` SDK, pinned to 1.8.0) exposing document-processing tools.

- `main.py` creates the `FastMCP("docs")` server and registers tools. Only functions explicitly registered here are exposed — a function in `tools/` is not an MCP tool until it is added with `mcp.tool()(fn)`.
- `tools/` holds plain Python functions, one module per domain:
  - `tools/math.py` — `add`, the reference example of a fully documented tool.
  - `tools/document.py` — `binary_document_to_markdown`, which converts DOCX/PDF bytes to markdown via `markitdown` (`StreamInfo(extension=...)` tells it the format). It is currently **not** registered in `main.py` and lacks tool-style docs/`Field` annotations.
- `tests/` tests the tool functions directly (not through the MCP layer), using binary fixtures in `tests/fixtures/` (`mcp_docs.docx`, `mcp_docs.pdf`).

## Defining MCP tools

Write the tool as a function in the appropriate `tools/` module, then register it in `main.py`:

```python
from tools.math import add
mcp.tool()(add)
```

FastMCP builds the tool's schema from the function signature and docstring, so both are what the AI client sees:

- Describe every parameter with pydantic `Field(description=...)` as the default value, and add type hints including the return type.
- The docstring should:
  - begin with a one-line summary,
  - give a detailed explanation of what the tool does,
  - explain when to use (and when not to use) the tool,
  - include usage examples with expected input/output.

```python
from pydantic import Field

def my_tool(
    param1: str = Field(description="Detailed description of this parameter"),
    param2: int = Field(description="Explain what this parameter does"),
) -> ReturnType:
    """Comprehensive docstring here"""
```

Follow `tools/math.py::add` as the template.
