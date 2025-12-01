# JWT Testing Guide

This guide provides curl commands and examples to test the JWT authentication implementation.

## Prerequisites
- Application running on `http://localhost:8080`
- curl command-line tool installed

## 1. Register a New User

```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@example.com",
    "password": "securePassword123"
  }'
```

**Expected Response (201 Created):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlcyI6W3siYXV0aG9yaXR5IjoiUk9MRV9URU5BTlQifV0sInN1YiI6ImpvaG4uZG9lQGV4YW1wbGUuY29tIiwiaWF0IjoxNzMyNDI3NjAwLCJleHAiOjE3MzI1MTQwMDB9.abc123...",
  "type": "Bearer",
  "username": "john.doe@example.com",
  "email": "john.doe@example.com"
}
```

## 2. Login with Credentials

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "password": "securePassword123"
  }'
```

**Expected Response (200 OK):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "type": "Bearer",
  "username": "john.doe@example.com",
  "email": "john.doe@example.com"
}
```

**Save the token from the response for the next steps.**

## 3. Get Authenticated User Info

Replace `YOUR_JWT_TOKEN` with the token received from login/register:

```bash
curl -X GET http://localhost:8080/api/auth/get-authenticated-user \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Expected Response (200 OK):**
```json
{
  "publicId": "550e8400-e29b-41d4-a716-446655440000",
  "email": "john.doe@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "imageUrl": null,
  "authorities": [
    "ROLE_TENANT"
  ]
}
```

## 4. Login Failure Test

Try logging in with wrong password:

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "password": "wrongPassword"
  }'
```

**Expected Response (401 Unauthorized):**
```json
{
  "error": "Invalid email or password"
}
```

## 5. Register Duplicate Email Test

Try registering with an email that already exists:

```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "Jane",
    "lastName": "Doe",
    "email": "john.doe@example.com",
    "password": "anotherPassword123"
  }'
```

**Expected Response (400 Bad Request):**
```json
{
  "error": "Email already registered"
}
```

## 6. Access Protected Endpoint Without Token

Try accessing a protected endpoint without providing a token:

```bash
curl -X GET http://localhost:8080/api/auth/get-authenticated-user
```

**Expected Response (401 Unauthorized):**
```
Unauthorized - No authentication token provided
```

## 7. Access Protected Endpoint With Invalid Token

```bash
curl -X GET http://localhost:8080/api/auth/get-authenticated-user \
  -H "Authorization: Bearer invalid.token.here"
```

**Expected Response (401 Unauthorized):**
```
Unauthorized - Invalid token
```

## 8. Logout

```bash
curl -X POST http://localhost:8080/api/auth/logout \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Expected Response (200 OK):**
```
(empty response body)
```

Note: JWT is stateless, so logout is mainly for frontend cleanup. The token remains valid until expiration.

## 9. Test Public Endpoints (No Token Required)

These endpoints should work without authentication:

```bash
# Get all listings by category
curl -X GET "http://localhost:8080/api/tenant-listing/get-all-by-category"

# Get single listing
curl -X GET "http://localhost:8080/api/tenant-listing/get-one?id=1"

# Search listings
curl -X POST http://localhost:8080/api/tenant-listing/search \
  -H "Content-Type: application/json" \
  -d '{"keyword": "beach"}'

# Check availability
curl -X GET "http://localhost:8080/api/booking/check-availability?listingId=1&startDate=2024-01-01&endDate=2024-01-10"
```

## 10. H2 Database Console

Access the H2 database console for debugging:

**URL:** http://localhost:8080/h2-console

**Connection Details:**
- JDBC URL: `jdbc:h2:mem:testdb`
- Username: `sa`
- Password: (leave empty)

Query example:
```sql
SELECT * FROM airbnb_user;
SELECT * FROM authority;
SELECT * FROM user_authority;
```

---

## JWT Token Structure

A JWT token consists of three parts separated by dots:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlcyI6W3siYXV0aG9yaXR5IjoiUk9MRV9URU5BTlQifV0sInN1YiI6ImpvaG4uZG9lQGV4YW1wbGUuY29tIiwiaWF0IjoxNzMyNDI3NjAwLCJleHAiOjE3MzI1MTQwMDB9.abc123...
│                                       │                                                                                                                          │
├─ Header (Base64)                      ├─ Payload (Base64)                                                                                                          ├─ Signature (HMAC-SHA256)
```

### Decoded Header:
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Decoded Payload:
```json
{
  "roles": [
    {
      "authority": "ROLE_TENANT"
    }
  ],
  "sub": "john.doe@example.com",
  "iat": 1732427600,
  "exp": 1732514000
}
```

## Postman Collection

If using Postman, create a collection with these requests:

1. **Register**
   - Method: POST
   - URL: `http://localhost:8080/api/auth/register`
   - Body: JSON with firstName, lastName, email, password

2. **Login**
   - Method: POST
   - URL: `http://localhost:8080/api/auth/login`
   - Body: JSON with email, password
   - Set environment variable: `{{token}}` to the response token

3. **Get User**
   - Method: GET
   - URL: `http://localhost:8080/api/auth/get-authenticated-user`
   - Header: `Authorization: Bearer {{token}}`

4. **Logout**
   - Method: POST
   - URL: `http://localhost:8080/api/auth/logout`
   - Header: `Authorization: Bearer {{token}}`

## Troubleshooting

### Token Expired
If you get an error about token expiration, the JWT token's 24-hour expiration has passed.
- Solution: Re-login to get a fresh token

### Invalid Secret Key
If tokens are not being validated correctly:
- Check that the secret key in `application.yml` matches
- Restart the application after changing the secret key

### Database Issues
If you encounter database errors:
- Check the H2 console at http://localhost:8080/h2-console
- Verify schema is created properly via Liquibase changelogs

### Role-Based Access Denied
If you get "Access Denied" errors:
- Verify your user has the required role
- Check `@PreAuthorize` annotations on the endpoint
- Use the "Get User" endpoint to verify your current authorities
