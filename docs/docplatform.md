# DocPlatform API

!!! note "Portfolio Sample"
    This page demonstrates how I design and document an API from the specification stage.
    The DocPlatform API is a fictional REST API. I authored the OpenAPI 3.0 spec by hand,
    without code generation, as a documentation architecture exercise. 
    
    The API subject matter is intentionally self-referential: a documentation management 
    platform, documented to the same standard I would apply to a production API.

The DocPlatform API is a fictional REST API designed and documented as a portfolio demonstration.
It models a documentation management platform where teams can create content, organize it into
versioned collections, manage contributors, and search across the full content library.

The spec below reflects the same standards I apply to production API documentation: consistent
error contracts, reusable component schemas, realistic pagination, and developer-facing prose
that goes beyond what the spec alone can communicate.

---

## Getting Started

### Base URL

All endpoints are relative to:

```
https://api.docplatform.io/v1
```

A sandbox environment is available for testing without affecting live data:

```
https://sandbox.api.docplatform.io/v1
```

### Authentication

Every request requires a Bearer token in the `Authorization` header.

```http
GET /documents HTTP/1.1
Host: api.docplatform.io
Authorization: Bearer <your_token>
```

Tokens are scoped to a role. The role determines which endpoints and actions are available.

| Role | Permissions |
|---|---|
| `admin` | Full read/write access across all resources |
| `author` | Create and update documents; read collections and contributors |
| `reviewer` | Read all resources; cannot create or modify |
| `viewer` | Read published documents and collections only |

A missing or invalid token returns `401 Unauthorized`. An expired token returns the same response
— your client should handle re-authentication transparently.

---

## Rate Limiting

Requests are limited to **1,000 per hour** per API token. Every response includes the following
headers so your client can track consumption:

| Header | Description |
|---|---|
| `X-RateLimit-Limit` | Maximum requests allowed per hour |
| `X-RateLimit-Remaining` | Requests remaining in the current window |
| `X-RateLimit-Reset` | Unix timestamp when the limit resets |

When the limit is exceeded, the API returns `429 Too Many Requests`. Back off and retry after
the `X-RateLimit-Reset` time.

---

## Errors

All errors follow a consistent envelope. Clients should key on `error.code` for programmatic
handling and surface `error.message` to end users.

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "One or more fields failed validation.",
    "details": [
      {
        "field": "slug",
        "issue": "Must contain only lowercase letters, numbers, and hyphens."
      }
    ]
  }
}
```

The `details` array is populated for `400` and `422` responses where field-level context is
available. It is omitted for `401`, `404`, and `429` responses.

### Error Code Reference

| HTTP Status | Code | When it occurs |
|---|---|---|
| `400` | `BAD_REQUEST` | Request body is malformed or missing required fields |
| `401` | `UNAUTHORIZED` | Token is missing, invalid, or expired |
| `404` | `NOT_FOUND` | The requested resource ID does not exist |
| `422` | `VALIDATION_ERROR` | Request is well-formed but fails a validation rule |
| `429` | `RATE_LIMIT_EXCEEDED` | Token has exceeded 1,000 requests per hour |

---

## Pagination

All list endpoints return paginated results using `page` and `per_page` query parameters.
The response envelope always includes `total` so clients can calculate page count without
an additional request.

**Request**

```http
GET /documents?page=2&per_page=10
```

**Response envelope**

```json
{
  "page": 2,
  "per_page": 10,
  "total": 84,
  "data": [ ... ]
}
```

The default page size is `20`. The maximum is `100`. Requests beyond the last page return
an empty `data` array — not a `404`.

---

## Versioning

This is **v1** of the DocPlatform API. The version is expressed in the base URL path rather
than a request header, making it explicit in every call and easy to support multiple versions
simultaneously.

Breaking changes are introduced only in new major versions. Non-breaking additions (new optional
fields, new endpoints) may be made to the current version at any time. When an endpoint is
deprecated, responses will include a `Deprecation` header with the scheduled `Sunset` date,
giving consumers adequate migration time.

### Changelog

| Version | Date | Notes |
|---|---|---|
| 1.2.0 | 2026-03-01 | Added `GET /collections/{id}/export` endpoint |
| 1.1.0 | 2026-01-15 | Added `role` filter parameter to `GET /contributors` |
| 1.0.0 | 2025-11-01 | Initial release |

---

## Interactive Reference

The interactive reference below is rendered directly from the OpenAPI 3.0 spec.
Use it to explore request parameters, response schemas, and example payloads for
every endpoint.

!!swagger docplatform.yaml!!

<style>
#swagger-ui-1 {
  background-color: white;
}
</style>
