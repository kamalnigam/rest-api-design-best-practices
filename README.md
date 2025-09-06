# REST API Design Best Practices & Rules Guide

*A comprehensive guide for experienced engineering teams*

## Table of Contents
1. [URI Design Principles](#uri-design-principles)
2. [HTTP Method Usage](#http-method-usage)
3. [Response Status Codes](#response-status-codes)
4. [Resource Modeling](#resource-modeling)
5. [Request/Response Design](#requestresponse-design)
6. [Error Handling](#error-handling)
7. [Security Best Practices](#security-best-practices)
8. [Performance & Caching](#performance--caching)
9. [Versioning Strategy](#versioning-strategy)
10. [Documentation](#documentation)
11. [Real-World Examples](#real-world-examples)
12. [Modern Enhancements and Trends](#modern-enhancements-and-trends)

---

## URI Design Principles

### Core URI Rules

#### **RULE 1: Use Forward Slash (/) for Hierarchical Relationships**
✅ **DO:**
```
https://api.company.com/v1/users/123/orders/456/items
https://api.github.com/repos/owner/repo/issues
```
❌ **DON'T:**
```
https://api.company.com/v1/users-orders-items
https://api.company.com/v1/getUserOrderItems
```

#### **RULE 2: No Trailing Forward Slash**
✅ **DO:**
```
https://api.company.com/v1/users
https://api.company.com/v1/users/123
```
❌ **DON'T:**
```
https://api.company.com/v1/users/
https://api.company.com/v1/users/123/
```

#### **RULE 3: Use Hyphens (-), Avoid Underscores (_)**
✅ **DO:**
```
https://api.company.com/v1/user-profiles
https://api.company.com/v1/order-items
```
❌ **DON'T:**
```
https://api.company.com/v1/user_profiles
https://api.company.com/v1/order_items
```
*Underscores can be obscured by text underlining in browsers and documentation.*

#### **RULE 4: Use Lowercase Letters**
✅ **DO:**
```
https://api.company.com/v1/products
https://api.company.com/v1/user-sessions
```
❌ **DON'T:**
```
https://api.company.com/v1/Products
https://api.company.com/v1/User-Sessions
```

#### **RULE 5: No File Extensions in URIs**
✅ **DO:**
```
https://api.company.com/v1/users/123
Accept: application/json
```
❌ **DON'T:**
```
https://api.company.com/v1/users/123.json
https://api.company.com/v1/users/123.xml
```
*Use HTTP headers for content negotiation instead.*

### Resource Naming Conventions

#### **RULE 6: Collections Use Plural Nouns**
✅ **DO:**
```
GET /users           # Collection of users
GET /products        # Collection of products
GET /orders          # Collection of orders
```

#### **RULE 7: Documents Use Singular Nouns**
✅ **DO:**
```
GET /users/123       # Single user document
GET /products/abc    # Single product document
```

#### **RULE 8: Controllers Use Verbs**
✅ **DO:**
```
POST /users/123/activate
POST /orders/456/cancel
POST /products/789/reindex
```
*For operations that don't fit standard CRUD operations.*

---

## HTTP Method Usage

### Standard HTTP Methods

#### **GET - Retrieve Resources**
```http
GET /users                    # Get all users
GET /users/123                # Get specific user
GET /users/123/orders         # Get user's orders
GET /orders?status=pending    # Get filtered orders
```
- Safe (no side effects)
- Idempotent (multiple calls = same result)
- Cacheable

#### **POST - Create Resources**
```http
POST /users              # Create new user
POST /orders             # Create new order
POST /users/123/activate # Execute controller action
```
**Example:**
```http
POST /users HTTP/1.1
Content-Type: application/json

{
  "email": "john@example.com",
  "name": "John Doe"
}
```
Response:
```http
HTTP/1.1 201 Created
Location: /users/124
Content-Type: application/json

{
  "id": 124,
  "email": "john@example.com",
  "name": "John Doe",
  "createdAt": "2025-01-15T10:00:00Z"
}
```

#### **PUT - Update/Replace Resources**
```http
PUT /users/123           # Update/replace entire user
PUT /configs/app-config  # Create/replace configuration
```
- Idempotent
- Replaces entire resource
- Creates resource if ID doesn't exist

#### **PATCH - Partial Updates**
```http
PATCH /users/123         # Partially update user
PATCH /orders/456        # Update specific order fields
```
Example:
```http
PATCH /users/123 HTTP/1.1
Content-Type: application/json

{
  "status": "inactive",
  "lastLogin": "2025-01-15T10:00:00Z"
}
```

#### **DELETE - Remove Resources**
```http
DELETE /users/123        # Delete user
DELETE /orders/456       # Delete order
```
- Idempotent
- Resource becomes unavailable after successful deletion

#### **HEAD - Retrieve Metadata**
```http
HEAD /users/123 HTTP/1.1
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 256
Last-Modified: Wed, 15 Jan 2025 10:00:00 GMT
ETag: "abc123"
```

#### **OPTIONS - Discover Capabilities**
```http
OPTIONS /users/123 HTTP/1.1
HTTP/1.1 200 OK
Allow: GET, PUT, PATCH, DELETE
Accept: application/json, application/xml
```

### Method Usage Matrix
| Resource Type | GET | POST | PUT | PATCH | DELETE |
|---------------|-----|------|-----|--------|--------|
| Collection    | ✅ List | ✅ Create | ❌ | ❌ | ❌ |
| Document      | ✅ Read | ❌ | ✅ Replace | ✅ Update | ✅ Delete |
| Controller    | ✅ Info | ✅ Execute | ❌ | ❌ | ❌ |

---

## Response Status Codes

### Success Codes (2xx)
- **200 OK**: Standard success
- **201 Created**: Resource created
- **202 Accepted**: Asynchronous processing
- **204 No Content**: Success without body

### Redirection Codes (3xx)
- **301 Moved Permanently**
- **304 Not Modified**

### Client Error Codes (4xx)
- **400 Bad Request**
- **401 Unauthorized**
- **403 Forbidden**
- **404 Not Found**
- **405 Method Not Allowed**
- **409 Conflict**
- **422 Unprocessable Entity**

### Server Error Codes (5xx)
- **500 Internal Server Error**
- **503 Service Unavailable**

Examples for all above codes are provided in the full guide.

---

## Resource Modeling

### Resource Archetypes

- **Document**: `/users/123`
- **Collection**: `/users`
- **Store**: `/users/123/favorites`
- **Controller**: `/users/123/activate`

### Resource Relationships
Hierarchical: `/companies/123/departments/456/employees/789`

Associated: `/users/123/orders`

Nested: (Max 3 levels recommended)

---

## Request/Response Design

### Content Types
- Use JSON as primary format.
- Support content negotiation via `Accept` header.

### Request Body Design
- Use camelCase for property names.
- Avoid abbreviations.

### Response Body Design
- Consistent structure with `data`, `meta`, and `links`.

### Filtering, Sorting, and Pagination
- Support filtering (`?status=active`)
- Support sorting (`?sort=lastName`)
- Support pagination (`?limit=20&offset=40`)

### Field Selection
- Use sparse fieldsets (`?fields=id,name,email`)

---

## Error Handling

### Standard Error Format
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains invalid data",
    "details": [
      { "field": "email", "code": "INVALID_FORMAT", "message": "Email format is invalid" },
      { "field": "age", "code": "OUT_OF_RANGE", "message": "Age must be between 18 and 120" }
    ],
    "timestamp": "2025-01-15T10:00:00Z",
    "path": "/api/v1/users",
    "traceId": "abc-123-def-456"
  }
}
```

Error code categories and scenarios are detailed in the guide.

---

## Security Best Practices

- Authenticate using API keys, JWT, or OAuth2.
- Use security headers (Strict-Transport-Security, Content-Security-Policy, etc).
- Validate and sanitize all input.
- Implement rate limiting.
- Mask sensitive information in responses.

---

## Performance & Caching

- Use proper Cache-Control headers.
- Enable response compression.
- Support resource expansion (`?expand=profile,orders`)
- Avoid N+1 queries; support batch operations.

---

## Versioning Strategy

- Use path-based versioning: `/api/v1/...`
- Avoid breaking changes; introduce new versions when necessary.
- Consider using media type versioning via Accept headers for advanced control.
- Document deprecation and migration paths for older versions.

---

## Documentation

- Use OpenAPI/Swagger for API documentation.
- Provide usage examples, error codes, and response schemas.
- Maintain interactive API explorers for easier onboarding.
- Include changelogs and migration guides.

---

## Real-World Examples

- [GitHub REST API](https://docs.github.com/en/rest)
- [Stripe API](https://stripe.com/docs/api)
- [Twilio API](https://www.twilio.com/docs/usage/api)
- [Slack API](https://api.slack.com/web)

---

## Modern Enhancements and Trends

### GraphQL and gRPC
- Consider GraphQL for flexible queries and reduced over-fetching.
- Use gRPC for high-performance, strongly-typed communication (especially for microservices).

### API Gateways & Observability
- Employ API gateways for routing, rate limiting, and security.
- Use distributed tracing, logging, and monitoring for API health and debugging.
  - Tools: OpenTelemetry, Prometheus, Grafana

### Hypermedia (HATEOAS)
- Provide actionable links in responses for improved discoverability and client navigation.

### Developer Experience
- Provide sandbox environments and mock servers for integration testing.
- Offer SDKs in popular languages for easier client integration.

---

> **Tip:** Follow these best practices to design scalable, secure, and maintainable REST APIs. Stay updated with trends in API development to continuously improve your systems.