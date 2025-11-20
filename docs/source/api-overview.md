# API Overview

The Query Disambiguation service exposes a RESTful API for analyzing and disambiguating natural language queries.

## Base URL

The API is served from the root of the application. All endpoint paths are relative to the base URL where the service is deployed.

**Base URL:** [https://datagems-dev.scayle.es/dg-query-disambiguation/](https://datagems-dev.scayle.es/dg-query-disambiguation/)

## Authentication

Currently, the API endpoints do not require authentication. See the [Security](./security.md) page for more details. The `/health` endpoint is publicly accessible.

---

## Endpoints

### Query Disambiguation
<p id="query-disambiguation"></p>
Analyzes a natural language query for ambiguity and uncertainty, providing general notes, ambiguous parts with rephrased options, overall clarity assessment, and suggested complete queries.

*   **Endpoint:** `POST /query_disambiguation`
*   **Description:** Takes a natural language query and analyzes it for ambiguities, especially focusing on spatial and temporal information. Returns structured analysis with replacement options and suggested queries.
*   **Request Body:**
    ```json
    {
      "query": "Show me the average temperature from some cities in Switzerland in the last month",
      "current_datetime": null
    }
    ```
    - `query` (required): The natural language query to analyze (string, 1-10000 characters)
    - `current_datetime` (optional): Current date and time string. If not provided, will use the current system time.

*   **Response Body (Success):**
    ```json
    {
      "result": {
        "general_notes": [
          {
            "aspect": "spatial",
            "description": "The query specifies 'some cities in Switzerland' without identifying which cities are included. Switzerland has multiple cities, and the lack of specificity makes it unclear which data is being requested.",
            "severity": "high"
          },
          {
            "aspect": "temporal",
            "description": "The query uses the term 'last month' which is a relative time reference. Based on the current date (2025-11-20), 'last month' refers to October 2025.",
            "severity": "medium"
          }
        ],
        "ambiguous_parts": [
          {
            "original_text": "some cities in Switzerland",
            "ambiguity_type": "spatial",
            "reason": "The phrase 'some cities in Switzerland' is vague because it does not specify which cities are being referred to. Switzerland has many cities, and the query does not clarify which ones are included.",
            "rephrased_options": [
              "Zurich, Geneva, and Bern",
              "major cities in Switzerland",
              "specific cities like Lausanne and Basel"
            ]
          },
          {
            "original_text": "last month",
            "ambiguity_type": "temporal",
            "reason": "The term 'last month' is a relative time reference that needs to be converted to a specific time period. Based on the current date, it should refer to October 2025.",
            "rephrased_options": [
              "October 2025",
              "2025-10"
            ]
          }
        ],
        "overall_clarity": "medium",
        "suggested_queries": [
          "Show me the average temperature from Zurich, Geneva, and Bern in Switzerland in October 2025",
          "Show me the average temperature from major cities in Switzerland in October 2025",
          "Show me the average temperature from specific cities like Lausanne and Basel in Switzerland in October 2025",
          "Show me the average temperature from Zurich, Geneva, and Bern in Switzerland in 2025-10",
          "Show me the average temperature from major cities in Switzerland in 2025-10",
          "Show me the average temperature from specific cities like Lausanne and Basel in Switzerland in 2025-10"
        ]
      }
    }
    ```

*   **Example Request (curl):**
    ```bash
    curl -X POST "https://datagems-dev.scayle.es/dg-query-disambiguation/query_disambiguation" \
      -H "Content-Type: application/json" \
      -d '{
        "query": "Show me the average temperature from some cities in Switzerland in the last month"
      }'
    ```

*   **Example Request (Python):**
    ```python
    import requests

    response = requests.post(
        "https://datagems-dev.scayle.es/dg-query-disambiguation/query_disambiguation",
        json={
            "query": "Show me the average temperature from some cities in Switzerland in the last month"
        }
    )
    result = response.json()
    print(result)
    ```

### Health Check
<p id="health"></p>
Verifies the operational status of the API.

*   **Endpoint:** `GET /health`
*   **Description:** Checks the availability of the API service. Returns a `200 OK` if the service is healthy. This endpoint does not require authentication.
*   **Response Body (Success):**
    ```json
    {
      "status": "healthy"
    }
    ```

### Root Endpoint
<p id="root"></p>
Returns basic information about the service.

*   **Endpoint:** `GET /`
*   **Description:** Returns service name, version, and documentation links.
*   **Response Body (Success):**
    ```json
    {
      "service": "Query Disambiguation Service",
      "version": "0.0.1",
      "docs": "/swagger"
    }
    ```
