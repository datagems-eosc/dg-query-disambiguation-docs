# System Architecture

The Query Disambiguation Service is a self-contained application designed to analyze and disambiguate natural language queries within the DataGEMS ecosystem.

## Core Components

1. **FastAPI Application:** The heart of the service is a Python web application built with FastAPI. It exposes the REST API, handles incoming HTTP requests, and orchestrates all internal operations.

2. **Query Disambiguation Tool:** A specialized module that uses LangChain and OpenAI LLM to analyze natural language queries for ambiguity. It identifies:

   - General uncertainty notes (spatial, temporal, entity, scope)
   - Specific ambiguous parts with their types
   - Replacement phrases for ambiguous parts
   - Complete suggested queries

3. **Date/Time Resolution:** A component that automatically resolves relative time references (e.g., "last month", "yesterday") based on the current system date/time, providing specific dates as replacement options.

4. **Output Parser:** Uses Pydantic models and LangChain's output parsers to ensure structured JSON responses that match the defined schema.

## Request Flow

### Query Disambiguation Flow

1. A user sends a `POST /query_disambiguation` request with a natural language query.
2. The service optionally receives a current date/time parameter, or uses the system's current date/time.
3. The Query Disambiguation Tool uses an LLM (via LangChain) to analyze the query for ambiguities.
4. The LLM identifies:
   - General notes about uncertainty (especially spatial and temporal)
   - Specific ambiguous parts with replacement phrase options
   - Overall clarity assessment
5. For relative time references, the service uses the current date/time to calculate specific dates.
6. The service generates suggested complete queries by combining replacement options.
7. Results are formatted as structured JSON and returned to the client.

## Technology Stack

- **FastAPI**: Modern, fast web framework for building APIs
- **LangChain**: Framework for developing applications powered by language models
- **OpenAI**: LLM provider for query analysis
- **Pydantic**: Data validation using Python type annotations
- **Uvicorn**: ASGI server for running the FastAPI application

## Data Models

The service uses Pydantic models for structured data:

- **QueryDisambiguationRequest**: Input request with query and optional current_datetime
- **QueryDisambiguationResult**: Output result containing:
  - `general_notes`: List of uncertainty notes
  - `ambiguous_parts`: List of ambiguous parts with rephrased options
  - `overall_clarity`: Categorical clarity level (low/medium/high)
  - `suggested_queries`: List of complete rephrased queries
