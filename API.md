# AI WeatherWise API

REST API reference for the AI WeatherWise backend.

## Base URL

```text
http://localhost:5000
```

Set the port from the `PORT` environment variable when the server is configured to use a different port.

## Request Conventions

- Send JSON request bodies with the `Content-Type: application/json` header.
- Protected endpoints require an access token in the `Authorization` header:

```text
Authorization: Bearer <jwt-token>
```

- IDs are MongoDB ObjectIds.
- Successful responses use `success: true`.
- Error responses use `success: false` and include a `message`.

## Endpoints

### Root

#### `GET /`

Returns a welcome message and a link to the API documentation.

**Response `200 OK`**

```json
{
  "message": "Welcome to the AI WeatherWise API!",
  "documentation": "Use Postman collection or check README.md for endpoint information"
}
```

---

## Authentication

### Register a user

#### `POST /api/auth/register`

Creates a user and returns a 30-day JWT.

**Request body**

```json
{
  "name": "Ganesh",
  "email": "ganesh@gmail.com",
  "password": "secret123"
}
```

**Response `201 Created`**

```json
{
  "success": true,
  "data": {
    "_id": "66f000000000000000000001",
    "name": "Ganesh",
    "email": "ganesh@gmail.com",
    "token": "<jwt-token>"
  }
}
```

**Errors**

- `400` if a field is missing, the password has fewer than 6 characters, or the email already exists.
- `500` if registration fails on the server.

**curl**

```bash
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Ganesh","email":"ganesh@gmail.com","password":"secret123"}'
```

### Log in

#### `POST /api/auth/login`

Authenticates a user and returns a 30-day JWT.

**Request body**

```json
{
  "email": "ganesh@gmail.com",
  "password": "secret123"
}
```

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "_id": "66f000000000000000000001",
    "name": "Ganesh",
    "email": "ganesh@gmail.com",
    "token": "<jwt-token>"
  }
}
```

**Errors**

- `400` if email or password is missing.
- `401` if the credentials are invalid.
- `500` if login fails on the server.

### Get the current user profile

#### `GET /api/auth/profile`

Requires authentication. Returns the authenticated user's profile without the password.

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "_id": "66f000000000000000000001",
    "name": "Ganesh",
    "email": "ganesh@gmail.com",
    "createdAt": "2026-09-23T10:00:00.000Z"
  }
}
```

**curl**

```bash
curl http://localhost:5000/api/auth/profile \
  -H "Authorization: Bearer <jwt-token>"
```

---

## Favorite Locations

All location endpoints require authentication. Locations belong to the authenticated user.

### Add a favorite location

#### `POST /api/locations`

**Request body**

```json
{
  "city": "London",
  "country": "United Kingdom"
}
```

**Response `201 Created`**

```json
{
  "success": true,
  "data": {
    "_id": "66f000000000000000000002",
    "city": "London",
    "country": "United Kingdom",
    "user": "66f000000000000000000001",
    "createdAt": "2026-09-23T10:05:00.000Z",
    "updatedAt": "2026-09-23T10:05:00.000Z"
  }
}
```

**Errors**

- `400` if city or country is missing, or the location is already a favorite.
- `401` if authentication is missing or invalid.
- `500` if the location cannot be saved.

### List favorite locations

#### `GET /api/locations`

Returns the authenticated user's locations, newest first.

**Response `200 OK`**

```json
{
  "success": true,
  "count": 1,
  "data": [
    {
      "_id": "66f000000000000000000002",
      "city": "London",
      "country": "United Kingdom",
      "user": "66f000000000000000000001",
      "createdAt": "2026-09-23T10:05:00.000Z",
      "updatedAt": "2026-09-23T10:05:00.000Z"
    }
  ]
}
```

### Update a favorite location

#### `PUT /api/locations/:id`

Replaces the city and country values for a location owned by the authenticated user.

**Request body**

```json
{
  "city": "Manchester",
  "country": "United Kingdom"
}
```

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "_id": "66f000000000000000000002",
    "city": "Manchester",
    "country": "United Kingdom",
    "user": "66f000000000000000000001",
    "createdAt": "2026-09-23T10:05:00.000Z",
    "updatedAt": "2026-09-23T10:10:00.000Z"
  }
}
```

**Errors**

- `400` if city or country is missing, or the update duplicates another favorite.
- `401` if authentication is missing, invalid, or the location belongs to another user.
- `404` if the location does not exist.
- `500` if the update fails.

### Delete a favorite location

#### `DELETE /api/locations/:id`

Deletes a location owned by the authenticated user.

**Response `200 OK`**

```json
{
  "success": true,
  "message": "Location removed from favorites",
  "data": {}
}
```

**Errors**

- `401` if authentication is missing, invalid, or the location belongs to another user.
- `404` if the location does not exist.
- `500` if deletion fails.

---

## Weather

### Get current weather

#### `GET /api/weather/:city`

Public endpoint. The city can contain spaces; URL-encode it when necessary.

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "city": "London",
    "temperature": 18.5,
    "humidity": 72,
    "windSpeed": 4.1,
    "condition": "Clouds",
    "isMock": false
  }
}
```

When `OPENWEATHER_API_KEY` is not configured, the API returns deterministic mock data with `isMock: true`.

**Errors**

- `404` if OpenWeatherMap cannot find the city.
- `500` if weather retrieval fails unexpectedly.

**curl**

```bash
curl http://localhost:5000/api/weather/London
```

---

## AI Weather Insights

Both endpoints require authentication. If `GEMINI_API_KEY` is not configured or Gemini fails, the service uses a local rule-based fallback.

### Generate a weather summary

#### `POST /api/ai/weather-summary`

**Request body**

```json
{
  "city": "London",
  "temperature": 18.5,
  "humidity": 72,
  "condition": "Cloudy"
}
```

`temperature` and `humidity` may be numbers or numeric strings. They are converted to numbers by the server.

**Response `200 OK`**

```json
{
  "success": true,
  "summary": "Today's weather in London is pleasant and cloudy with high humidity."
}
```

**Errors**

- `400` if city, temperature, humidity, or condition is missing.
- `401` if authentication is missing or invalid.
- `500` if summary generation fails.

### Generate weather recommendations

#### `POST /api/ai/weather-recommendation`

**Request body**

```json
{
  "temperature": 18.5,
  "condition": "Cloudy"
}
```

**Response `200 OK`**

```json
{
  "success": true,
  "recommendation": "Enjoy the comfortable temperature, great day for outdoor plans, and a light jacket might be handy."
}
```

**Errors**

- `400` if temperature or condition is missing.
- `401` if authentication is missing or invalid.
- `500` if recommendation generation fails.

---

## Common Errors

### Missing token

**Response `401 Unauthorized`**

```json
{
  "success": false,
  "message": "Not authorized, no token provided"
}
```

### Invalid token

**Response `401 Unauthorized`**

```json
{
  "success": false,
  "message": "Not authorized, token verification failed"
}
```

### Unknown route

**Response `404 Not Found`**

```json
{
  "success": false,
  "message": "Route not found: GET /api/unknown"
}
```

## Postman

Import `postman_collection.json` into Postman. Register or log in first; the collection stores the returned JWT in its `token` variable for protected requests.
