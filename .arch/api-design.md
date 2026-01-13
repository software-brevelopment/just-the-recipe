# API Design

## RESTful API Endpoints

### Core Extraction Endpoint

#### POST /api/v1/extract
Extract recipe information from a URL.

**Request Body:**
```json
{
  "url": "https://example.com/recipe",
  "options": {
    "includeImages": true,
    "includeNutrition": false,
    "normalizeUnits": true,
    "preferredLanguage": "en"
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Recipe Title",
    "ingredients": [...],
    "instructions": [...],
    // ... full recipe object
  },
  "processingTime": 1250,
  "warnings": ["Could not extract nutritional information"]
}
```

### Recipe Management

#### GET /api/v1/recipes/{id}
Retrieve a previously extracted recipe.

#### GET /api/v1/recipes/search
Search extracted recipes.

**Query Parameters:**
- `q`: Search query
- `cuisine`: Filter by cuisine
- `difficulty`: Filter by difficulty
- `limit`: Results per page (default: 20)
- `offset`: Pagination offset

#### DELETE /api/v1/recipes/{id}
Delete a stored recipe.

### Utility Endpoints

#### GET /api/v1/sites/supported
List of supported recipe sites with confidence scores.

#### POST /api/v1/validate
Validate if a URL contains extractable recipe content.

**Request Body:**
```json
{
  "url": "https://example.com/recipe"
}
```

**Response:**
```json
{
  "extractable": true,
  "confidence": 0.85,
  "site": "example.com",
  "estimatedProcessingTime": 2000
}
```

#### GET /api/v1/health
Health check endpoint.

## WebSocket API

### Real-time Extraction Updates

#### Connection: /ws/extract
Subscribe to real-time extraction progress.

**Message Format:**
```json
{
  "type": "extraction_progress",
  "data": {
    "url": "https://example.com/recipe",
    "stage": "parsing_html",
    "progress": 0.6,
    "message": "Extracting ingredients..."
  }
}
```

## Authentication

### API Key Authentication
```http
Authorization: Bearer your-api-key
```

### Rate Limiting
- **Free tier**: 100 requests/hour
- **Premium tier**: 1000 requests/hour
- **Enterprise**: Unlimited

Headers:
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
```

## Error Handling

### HTTP Status Codes
- `200 OK`: Successful request
- `201 Created`: Resource created
- `400 Bad Request`: Invalid input
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource not found
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server error

### Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "EXTRACTION_FAILED",
    "message": "Could not extract recipe from the provided URL",
    "details": {
      "url": "https://example.com/recipe",
      "reason": "No structured recipe data found"
    }
  },
  "requestId": "uuid"
}
```

### Error Codes
- `INVALID_URL`: Malformed URL
- `UNREACHABLE_URL`: Cannot fetch the URL
- `NO_RECIPE_FOUND`: No recipe content detected
- `EXTRACTION_FAILED`: Parsing failed
- `RATE_LIMIT_EXCEEDED`: Too many requests
- `SITE_NOT_SUPPORTED`: Unsupported recipe site
- `CONTENT_PROTECTED`: Content behind paywall/login

## Response Caching

### Cache-Control Headers
```http
Cache-Control: public, max-age=86400
ETag: "recipe-uuid-hash"
Last-Modified: Wed, 12 Jan 2026 19:43:00 GMT
```

### Conditional Requests
```http
If-None-Match: "recipe-uuid-hash"
If-Modified-Since: Wed, 12 Jan 2026 19:43:00 GMT
```

## API Versioning

### URL Versioning
- `/api/v1/` - Current stable version
- `/api/v2/` - Next version (breaking changes)

### Backward Compatibility
- Maintain v1 for at least 12 months after v2 release
- Deprecation warnings in response headers
- Migration guides for breaking changes

## OpenAPI Specification

### Schema Definition
```yaml
openapi: 3.0.0
info:
  title: Recipe Extractor API
  version: 1.0.0
  description: API for extracting recipe information from websites
paths:
  /api/v1/extract:
    post:
      summary: Extract recipe from URL
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ExtractionRequest'
      responses:
        '200':
          description: Successful extraction
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ExtractionResponse'
```

## SDK Examples

### JavaScript/TypeScript
```typescript
import { RecipeExtractor } from '@just-the-recipe/client';

const client = new RecipeExtractor({
  apiKey: 'your-api-key',
  baseUrl: 'https://api.just-the-recipe.com'
});

const recipe = await client.extract({
  url: 'https://example.com/recipe',
  options: { includeImages: true }
});
```

### Python
```python
from just_the_recipe import RecipeClient

client = RecipeClient(api_key='your-api-key')
recipe = client.extract(
    url='https://example.com/recipe',
    options={'include_images': True}
)
```
