# Cat Facts API

!!! note "Portfolio Sample"
    This page demonstrates how I structure and present API reference documentation
    for a developer audience. The Cat Facts API is an existing public API — I did not
    design or build it. The OpenAPI spec, surrounding documentation, and presentation
    decisions are my own work.

The Cat Facts API is a lightweight public REST API that returns information about domestic
cats: a list of known breeds and randomly selected cat facts. It requires no authentication
and has no write operations, making it a clean subject for demonstrating API reference
documentation structure and presentation.

---

## Base URL

All endpoints are relative to:

```
https://catfact.ninja
```

The API serves over HTTPS only. HTTP requests are not supported.

---

## Authentication

The Cat Facts API requires no authentication. All endpoints are publicly accessible
without a token or API key.

---

## Requests and Responses

The API accepts standard HTTP `GET` requests. No request body is required for any endpoint.
All responses are returned as `application/json`.

### Query Parameters

Optional query parameters are used to control result sets on list endpoints. Parameters
are passed as URL query strings:

```http
GET /breeds?limit=10&page=2
```

Unrecognized parameters are silently ignored.

### Response Format

Single-resource endpoints return a JSON object. List endpoints return a paginated envelope:

```json
{
  "current_page": 1,
  "data": [ ... ],
  "first_page_url": "https://catfact.ninja/breeds?page=1",
  "last_page": 5,
  "last_page_url": "https://catfact.ninja/breeds?page=5",
  "next_page_url": "https://catfact.ninja/breeds?page=2",
  "prev_page_url": null,
  "per_page": 25,
  "to": 25,
  "total": 98
}
```

When you are on the last page, `next_page_url` returns `null`. Check this field in your
client rather than relying on `total` to determine when to stop paginating.

---

## Endpoints at a Glance

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/breeds` | Returns a paginated list of cat breeds |
| `GET` | `/fact` | Returns a single random cat fact |
| `GET` | `/facts` | Returns a paginated list of cat facts |

---

## Common Parameters

These parameters are shared across list endpoints (`/breeds` and `/facts`).

| Parameter | Type | Default | Description |
|---|---|---|---|
| `limit` | integer | 25 | Number of results per page. Accepted range: 1–1000. |
| `page` | integer | 1 | Page number to retrieve (1-based). |

### The `max_length` Parameter

The `/fact` and `/facts` endpoints accept an additional `max_length` parameter that
limits results to facts at or below a specified character count. This is useful when
displaying facts in space-constrained UI contexts such as push notifications or cards.

```http
GET /fact?max_length=140
```

If no facts exist within the specified length, the API returns an empty result rather
than an error.

---

## Schemas

### Breed

Represents a single cat breed with physical characteristics and origin data.

| Field | Type | Description |
|---|---|---|
| `breed` | string | Common breed name |
| `country` | string | Country of origin |
| `origin` | string | Geographic or cultural origin detail |
| `coat` | string | Coat type (e.g., short, long, hairless) |
| `pattern` | string | Coat pattern (e.g., tabby, solid, colorpoint) |

### CatFact

Represents a single cat fact entry.

| Field | Type | Description |
|---|---|---|
| `fact` | string | The cat fact text |
| `length` | integer | Character count of the fact string |

---

## Usage Examples

### Get a random fact

```http
GET /fact HTTP/1.1
Host: catfact.ninja
```

```json
{
  "fact": "Cats have five toes on each front paw, but only four on the back paws.",
  "length": 70
}
```

### Get a short fact for a notification

```http
GET /fact?max_length=80 HTTP/1.1
Host: catfact.ninja
```

### Get the second page of breeds

```http
GET /breeds?limit=10&page=2 HTTP/1.1
Host: catfact.ninja
```

---

## Interactive Reference

The interactive reference below is rendered from the OpenAPI 3.0 JSON spec.
Expand each endpoint to explore parameters, response schemas, and example payloads.

!!swagger openapi.json!!

<style>
#swagger-ui-1 {
  background-color: white;
}
</style>



