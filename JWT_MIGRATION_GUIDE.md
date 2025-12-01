# OAuth2/Auth0 to JWT Migration Summary

## Overview
Successfully migrated your Airbnb Clone backend from OAuth2/Auth0 authentication to JWT (JSON Web Token) with role-based security, and replaced PostgreSQL with H2 in-memory database.

---

## Changes Made

### 1. **Dependencies (pom.xml)**
- ✅ Removed: `spring-boot-starter-oauth2-client`
- ✅ Removed: `okta-spring-boot-starter` (Okta dependency)
- ✅ Removed: `com.auth0:auth0` (Auth0 client library)
- ✅ Removed: `org.postgresql:postgresql`
- ✅ Added: `io.jsonwebtoken:jjwt-api`, `jjwt-impl`, `jjwt-jackson` (JWT library)
- ✅ Added: `com.h2database:h2` (H2 in-memory database)

### 2. **Database Configuration (application.yml)**
- ✅ Changed datasource from PostgreSQL to H2
- ✅ Added H2 console for debugging (at `/h2-console`)
- ✅ Removed: All OAuth2/Auth0 configuration
- ✅ Added: JWT secret key and expiration settings
  - JWT Token Expiration: 86400000 ms (24 hours)
  - Refresh Token Expiration: 604800000 ms (7 days)

### 3. **Security Implementation**

#### New JWT Files Created:
- **`JwtTokenProvider.java`** - Handles JWT token generation, validation, and claims extraction
  - Methods: `generateToken()`, `isTokenValid()`, `extractUsername()`, `extractAllClaims()`
  - Uses HMAC-SHA algorithm with configurable secret key

- **`JwtTokenFilter.java`** - Spring Security filter for JWT validation on every request
  - Extracts JWT from Authorization header (Bearer token)
  - Sets authentication in SecurityContext based on token claims
  - Automatically runs before each request

#### Updated Security Configuration:
- **`SecurityConfiguration.java`** - Replaced OAuth2 config with JWT-based security
  - Session policy: STATELESS (no session cookies needed)
  - CSRF protection: Disabled (not needed for JWT)
  - Added JWT filter to security chain
  - Role-based authorization: `@EnableMethodSecurity` for `@Secured`, `@RolesAllowed`, `@PreAuthorize`

### 4. **User Management**

#### User Entity Updated (`User.java`):
- ✅ Added `password` field (String, encrypted with BCrypt)
- ✅ Made email unique and not nullable
- ✅ Authorities relationship remains (for role-based access control)

#### New DTOs Created:
- **`LoginDTO`** - For login requests (email, password)
- **`RegisterDTO`** - For user registration (firstName, lastName, email, password)
- **`AuthResponseDTO`** - JWT token response (token, type, username, email)

#### UserService Updated:
- ✅ Added `register()` method - Creates new user with TENANT role by default
- ✅ Added `login()` method - Validates credentials and returns JWT token
- ✅ Added password encoding with BCryptPasswordEncoder
- ✅ Removed all OAuth2/IDP synchronization logic

#### AuthResource Updated:
- ✅ New `POST /api/auth/register` - User registration endpoint
- ✅ New `POST /api/auth/login` - User login endpoint
- ✅ Updated `GET /api/auth/get-authenticated-user` - Get authenticated user from JWT
- ✅ `POST /api/auth/logout` - Simple logout (JWT is stateless)
- ✅ Removed OAuth2-specific endpoints

### 5. **Database Schema**

#### Liquibase Changes (`00000000000001_user.xml`):
- ✅ Added `password` column to `airbnb_user` table (VARCHAR(255), NOT NULL)
- ✅ Changed email constraint from nullable to NOT NULL
- ✅ Schema version managed by Liquibase changelogs

### 6. **Utility Classes Updated**

#### SecurityUtils Simplified:
- ✅ Removed OAuth2 claims mapping (`mapOauth2AttributesToUser()`)
- ✅ Removed OIDC/OAuth2-specific authority extraction
- ✅ Kept core role constants: `ROLE_TENANT`, `ROLE_LANDLORD`
- ✅ Kept `hasCurrentUserAnyOfAuthorities()` for role checking

#### LandlordService Updated:
- ✅ Removed Auth0Service dependency
- ✅ Removed automatic role assignment logic (handled via user registration now)

### 7. **Removed Files**
- ✅ `Auth0Service.java` - No longer needed (deleted)
- ✅ All OAuth2/Okta configuration removed

### 8. **Docker Compose**
- ✅ `compose.yaml` - PostgreSQL service commented out (H2 is embedded in app)

---

## API Changes

### Authentication Endpoints

#### Register User
```bash
POST /api/auth/register
Content-Type: application/json

{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com",
  "password": "securePassword123"
}

Response (201 Created):
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "type": "Bearer",
  "username": "john@example.com",
  "email": "john@example.com"
}
```

#### Login User
```bash
POST /api/auth/login
Content-Type: application/json

{
  "email": "john@example.com",
  "password": "securePassword123"
}

Response (200 OK):
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "type": "Bearer",
  "username": "john@example.com",
  "email": "john@example.com"
}
```

#### Get Authenticated User
```bash
GET /api/auth/get-authenticated-user
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Response (200 OK):
{
  "publicId": "550e8400-e29b-41d4-a716-446655440000",
  "email": "john@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "imageUrl": null,
  "authorities": ["ROLE_TENANT"]
}
```

#### Logout
```bash
POST /api/auth/logout
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Response (200 OK)
```

### Using the Token
All authenticated endpoints require the JWT token in the Authorization header:
```
Authorization: Bearer <your_jwt_token_here>
```

---

## Role-Based Access Control

Default roles assigned:
- **ROLE_TENANT** - Default for new users (can book listings)
- **ROLE_LANDLORD** - Users who create listings

To restrict endpoints by role, use Spring Security annotations:
```java
@PreAuthorize("hasRole('LANDLORD')")
@PostMapping("/properties")
public ResponseEntity<CreatedListingDTO> createProperty(@RequestBody SaveListingDTO dto) {
    // Only users with ROLE_LANDLORD can access
}

@PreAuthorize("hasAnyRole('TENANT', 'LANDLORD')")
@GetMapping("/bookings")
public ResponseEntity<List<BookedListingDTO>> getBookings() {
    // Accessible to both tenants and landlords
}
```

---

## Configuration

### JWT Settings (`application.yml`)
```yaml
application:
  jwt:
    secret-key: mySecretKeyForJWTTokenGenerationAndValidationPurpose123456789
    expiration: 86400000        # 24 hours in milliseconds
    refresh-token-expiration: 604800000  # 7 days in milliseconds
```

### Database (H2)
- **URL**: jdbc:h2:mem:testdb
- **Console**: http://localhost:8080/h2-console
- **Username**: sa
- **Password**: (empty)

---

## Security Notes

⚠️ **Important for Production:**
1. **Change JWT Secret Key** - Use a strong, random secret key in production
2. **Use HTTPS** - Always use HTTPS in production to protect tokens in transit
3. **Store Token Securely** - On frontend, store JWT in secure HttpOnly cookies or secure storage
4. **Implement Token Refresh** - Consider implementing token refresh endpoints for long-lived sessions
5. **Add Rate Limiting** - Protect login/register endpoints from brute force attacks
6. **Validate Input** - All endpoints validate input with `@Valid` annotations
7. **Password Encoding** - Passwords are hashed with BCryptPasswordEncoder (10 rounds)

---

## Testing the Changes

### 1. Start the application
```bash
mvn spring-boot:run
```

### 2. Register a new user
```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "Test",
    "lastName": "User",
    "email": "test@example.com",
    "password": "password123"
  }'
```

### 3. Login
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123"
  }'
```

### 4. Use the token to access protected endpoints
```bash
curl -X GET http://localhost:8080/api/auth/get-authenticated-user \
  -H "Authorization: Bearer <your_token_here>"
```

---

## Migration Complete ✅

All OAuth2/Auth0 dependencies and configurations have been successfully removed and replaced with JWT-based authentication. The project now uses H2 in-memory database for development and local testing.
