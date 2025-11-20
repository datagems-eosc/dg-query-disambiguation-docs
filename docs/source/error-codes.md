# Status & Error Codes

The Query Disambiguation Service uses standard HTTP status codes to indicate the success or failure of API requests.

## Success Codes

### 200 OK
The request was successful.

**Example:**
```json
{
  "result": {
    "general_notes": [...],
    "ambiguous_parts": [...],
    "overall_clarity": "medium",
    "suggested_queries": [...]
  }
}
```

## Client Error Codes

### 400 Bad Request
The request was malformed or invalid.

**Common causes:**
- Missing required fields
- Invalid JSON format
- Query length exceeds maximum (10,000 characters)
- Invalid date/time format

**Example Response:**
```json
{
  "detail": "query field is required"
}
```

### 422 Unprocessable Entity
The request was well-formed but contains semantic errors.

**Common causes:**
- Validation errors in request body
- Invalid data types

**Example Response:**
```json
{
  "detail": [
    {
      "loc": ["body", "query"],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```

## Server Error Codes

### 500 Internal Server Error
An unexpected error occurred on the server.

**Common causes:**
- OpenAI API errors
- LLM processing failures
- Internal service errors

**Example Response:**
```json
{
  "detail": "An error occurred during disambiguation: [error message]"
}
```

## Error Response Format

All error responses follow this format:

```json
{
  "detail": "Error message describing what went wrong"
}
```

For validation errors (422), the detail may be an array of validation errors:

```json
{
  "detail": [
    {
      "loc": ["body", "field_name"],
      "msg": "Error message",
      "type": "error_type"
    }
  ]
}
```

## Handling Errors

When making API requests, always check the HTTP status code:

```python
import requests

response = requests.post(
    "https://datagems-eosc.github.io/dg-query-disambiguation/query_disambiguation",
    json={"query": "test query"}
)

if response.status_code == 200:
    result = response.json()
    # Process successful response
elif response.status_code == 400:
    print(f"Bad request: {response.json()['detail']}")
elif response.status_code == 500:
    print(f"Server error: {response.json()['detail']}")
else:
    print(f"Unexpected status: {response.status_code}")
```
