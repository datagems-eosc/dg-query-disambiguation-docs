# Developer Onboarding Guide

Welcome to the developer guide for the **In-Dataset Discovery Service**.

This document provides a comprehensive overview of the service's architecture and a step-by-step guide for extending it with new features, agents, and tools.

## Table of Contents

- [Developer Onboarding Guide](#developer-onboarding-guide)
  - [Table of Contents](#table-of-contents)
  - [1. Service Overview](#1-service-overview)
    - [Core Principles](#core-principles)
    - [Directory Structure](#directory-structure)
    - [Key Components](#key-components)
  - [2. System Architecture](#2-system-architecture)
    - [API Layer](#api-layer)
    - [Agent Layer](#agent-layer)
    - [Tool Layer](#tool-layer)
  - [3. Adding New Features](#3-adding-new-features)
    - [Adding a New API Endpoint](#adding-a-new-api-endpoint)
    - [Adding a New Tool](#adding-a-new-tool)
  - [4. Development Workflow](#4-development-workflow)
    - [Local Setup](#local-setup)
    - [Testing](#testing)
    - [Code Style](#code-style)
    - [Best Practices](#best-practices)
  - [Next Steps](#next-steps)

---

## 1. Service Overview

The In-Dataset Discovery Service is a FastAPI-based application that provides natural language interfaces for exploring datasets. It uses LLM capabilities to convert user questions into executable queries and agents to orchestrate complex multi-step tasks.

### Core Principles

The service is built on a few key principles:

- **Modularity:** Each component (agents, tools, models) is self-contained and can be extended independently.
- **LLM-Powered:** Leverages LLM and LangChain for intelligent query generation and processing.
- **Agent-Based:** Uses LangGraph to orchestrate complex workflows that combine multiple tools.
- **Type Safety:** Uses Pydantic models for request/response validation and type safety.

### Directory Structure

```
dg-query-disambiguation/
├── src/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py              # FastAPI application
│   │   ├── schemas.py           # Request/Response models
│   │   └── dependencies.py      # Dependency injection
│   └── tools/
│       ├── __init__.py
│       └── query_disambiguation.py  # Core disambiguation tool
├── main.py                      # Application entry point
├── example.py                   # Usage examples
├── pyproject.toml              # Project configuration
├── README.md                   # This file
└── .env                        # Environment variables (not in git)
```

### Key Components

1. **FastAPI Application** (`src/app/main.py`): Defines all API endpoints and request/response models.
2. **Tools** (`src/tools/`): Reusable functions that agents can call:
   - `query_disambiguation.py`: Tool for disambiguating natural language queries
---

## 2. System Architecture

### API Layer

The API layer (`src/app/main.py`) handles HTTP requests and responses. Each endpoint:

- Validates input using Pydantic models
- Handles authentication (if required) (not implemented yet)
- Calls the appropriate tool
- Returns structured responses

### Agent Layer

Agents are the core processing units that handle specific types of queries:

- **Query Disambiguation Agent**: Processes natural language queries and disambiguates them

### Tool Layer

Tools are reusable functions that agents can invoke. They encapsulate specific operations like:
- Disambiguating natural language queries

---

## 3. Adding New Features

### Adding a New API Endpoint

1. **Define Request/Response Models** in `src/app/main.py`:

```python
class MyQueryRequest(BaseModel):
    question: str = Field(..., description="The question to process")

class MyQueryResponse(BaseModel):
    answer: str
    metadata: Dict[str, Any]
```

2. **Create the Endpoint**:

```python
@app.post("/my-endpoint", response_model=MyQueryResponse)
async def my_endpoint(query: MyQueryRequest):
    """Process a query and return results."""
    # Your logic here
    result = process_query(query.question)
    return MyQueryResponse(answer=result, metadata={})
```

3. **Add Error Handling**:

```python
try:
    result = process_query(query.question)
    return MyQueryResponse(answer=result, metadata={})
except Exception as e:
    logger.error("query_failed", error=str(e))
    raise HTTPException(status_code=500, detail=str(e))
```

1. **Add Tools** (if needed) in `src/tools/`:

```python
# src/agents/tools/my_tool.py
def my_tool(param: str) -> Dict[str, Any]:
    """A tool that does something useful."""
    # Tool implementation
    return {"result": "value"}
```

3. **Integrate with API** in `src/app/main.py`:

```python
from src.agents.my_agent import my_agent

@app.get("/my-endpoint")
async def my_endpoint(question: str, log: structlog.BoundLogger = Depends(get_logger)):
    return my_agent(question, log)
```

### Adding a New Tool

1. **Create Tool File** in `src/tools/`:

```python
# src/agents/tools/my_tool.py
from typing import Dict, Any

def my_tool(param1: str, param2: int) -> Dict[str, Any]:
    """
    Tool description.

    Args:
        param1: Description of param1
        param2: Description of param2

    Returns:
        Dictionary with results
    """
    # Tool implementation
    return {"result": "value"}
```

---

## 4. Development Workflow

### Local Setup

1. **Clone the repository**:
```bash
git clone git@github.com:datagems-eosc/dg-query-disambiguation.git
cd dg-query-disambiguation
```

2. **Install dependencies**:
```bash
# Using uv (recommended)
uv sync

# Or using pip
pip install -e .
```

3. **Set up environment variables**:
```bash
cp .env.template .env
# Edit .env with your configuration
```

4. **Run the service**:
```bash
# Using uvicorn directly
uvicorn src.app.main:app --reload --port 8080

# Or using Docker
docker build -t dg-query-disambiguation .
docker run --env-file .env -p 8080:8080 dg-query-disambiguation
```

### Testing

1. **Run health check**:

```bash
curl https://datagems-eosc.github.io/dg-query-disambiguation/health
```

2. **Test an endpoint**:

```bash
curl -X POST "https://datagems-eosc.github.io/dg-query-disambiguation/query_disambiguation" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "Show me the data from that place last month"
  }'
```

3. **View API documentation**:

- Swagger UI: https://datagems-eosc.github.io/dg-query-disambiguation/swagger
- ReDoc: https://datagems-eosc.github.io/dg-query-disambiguation/redoc

### Code Style

- Follow PEP 8 style guidelines
- Use type hints for all function parameters and return values
- Use Pydantic models for data validation
- Add docstrings to all public functions and classes
- Use structured logging with `structlog`
- Handle errors gracefully and return appropriate HTTP status codes

### Best Practices

1. **Error Handling**: Always wrap agent calls in try-except blocks and log errors appropriately.
2. **Logging**: Use structured logging with context (user ID, correlation ID, etc.).
3. **Validation**: Use Pydantic models for all request/response validation.
4. **Security**: Follow the security patterns established in `src/app/security.py` for protected endpoints.
5. **Testing**: Write tests for new features and ensure they pass before submitting PRs.

---

## Next Steps

- Review the [Architecture](./architecture.md) documentation for more details on system design
- Check the [API Overview](./api-overview.md) to understand existing endpoints
- Read the [Configuration](./configuration.md) guide for environment setup
- Explore the source code in `src/agents/` to see examples of agent implementations
