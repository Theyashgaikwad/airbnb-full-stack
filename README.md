# Airbnb clone (fullstack project) Spring boot 3, Angular 17, PrimeNG, H2/PostgreSQL, JWT Authentication (2024) (Backend)

Spring boot backend of the airbnb clone

[Video tutorial](https://youtu.be/XriUV06Hkow)

[Angular Frontend](https://github.com/C0de-cake/airbnb-clone-frontend)

### Key Features:
- 📅 Booking management for travelers
- 🏠 Landlord reservation management
- 🔍 Search for houses by criteria (location, date, guests, beds, etc)
- 🔐 Authentication and Authorization (Role management) with JWT
- 🏢 Domain-driven design
- 💾 H2 in-memory database for development (PostgreSQL for production)

## Architecture Changes (JWT Migration)
- ✅ Migrated from OAuth2/Auth0 to JWT-based authentication
- ✅ Replaced PostgreSQL with H2 for development
- ✅ Role-based access control (ROLE_TENANT, ROLE_LANDLORD)
- ✅ Stateless session management

## Usage
### Prerequisites
- [JDK 21](https://adoptium.net/temurin/releases/)
- [Maven](https://maven.apache.org/download.cgi) or use bundled mvnw
- IDE ([VSCode](https://code.visualstudio.com/download), [IntelliJ](https://www.jetbrains.com/idea/download/))
- [PostgreSQL](https://www.postgresql.org/download/) - For production only

### Clone the repository
```bash
git clone https://github.com/C0de-cake/airbnb-clone-back
cd airbnb-clone-back
```

### Launch

#### Quick Start (Development with H2)
```bash
# Using Maven wrapper
./mvnw spring-boot:run

# Or on Windows
mvnw.cmd spring-boot:run
```

The application will start on `http://localhost:8080`

#### With Custom JWT Settings
```bash
./mvnw spring-boot:run -Dspring-boot.run.arguments="--application.jwt.secret-key=your-secret-key --application.jwt.expiration=86400000"
```

#### IntelliJ
1. Open project in IntelliJ
2. Find `AirbnbCloneBackApplication.java` in `src/main/java`
3. Right-click → Run

#### Production with PostgreSQL
```bash
./mvnw spring-boot:run -Dspring.profiles.active=prod \
  -Dspring-boot.run.arguments="--POSTGRES_USER=username --POSTGRES_PASSWORD=password --POSTGRES_URL=localhost --POSTGRES_DB=airbnb_clone"
```

### API Documentation

#### Authentication Endpoints
- `POST /api/auth/register` - Create new account
- `POST /api/auth/login` - Login and get JWT token
- `GET /api/auth/get-authenticated-user` - Get current user info
- `POST /api/auth/logout` - Logout

#### Example: Register User
```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "John",
    "lastName": "Doe",
    "email": "john@example.com",
    "password": "password123"
  }'
```

#### Example: Login
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john@example.com",
    "password": "password123"
  }'
```

### Database Access
Access H2 console at: `http://localhost:8080/h2-console`
- JDBC URL: `jdbc:h2:mem:testdb`
- Username: `sa`
- Password: (leave empty)

### Documentation Files
- `JWT_MIGRATION_GUIDE.md` - Detailed migration from OAuth2 to JWT
- `JWT_TESTING_GUIDE.md` - API testing examples
- `VERIFICATION_CHECKLIST.md` - Implementation checklist
- `SETUP_AND_RUN_INSTRUCTIONS.md` - Complete setup guide for full stack

