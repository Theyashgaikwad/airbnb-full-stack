# Airbnb clone (fullstack project) Spring boot 3, Angular 17, PrimeNG, H2/PostgreSQL, JWT Authentication (2024) (Frontend)

Angular frontend of the airbnb clone

[Video tutorial](https://youtu.be/XriUV06Hkow)

[Spring boot Backend](https://github.com/C0de-cake/airbnb-clone-backend)

### Key Features:
- 📅 Booking management for travelers
- 🏠 Landlord reservation management
- 🔍 Search for houses by criteria (location, date, guests, beds, etc)
- 🔐 Authentication and Authorization (Role management) with JWT
- 🏢 Domain-driven design
- 🎨 PrimeNG UI components

## Architecture Changes (JWT Migration)
- ✅ Migrated from OAuth0 to JWT-based authentication
- ✅ Updated API calls to use JWT Bearer tokens
- ✅ Token stored in browser storage
- ✅ Automatic token refresh on expiration (24 hours)

## Usage
### Prerequisites
- [NodeJS 20.11+ LTS](https://nodejs.org/)
- [Angular CLI v17](https://www.npmjs.com/package/@angular/cli)
- IDE ([VSCode](https://code.visualstudio.com/download), [IntelliJ](https://www.jetbrains.com/idea/download/))

### Installation

#### Clone the repository
```bash
git clone https://github.com/C0de-cake/airbnb-clone-frontend
cd airbnb-clone-frontend
```

#### Install Angular CLI (if not already installed)
```bash
npm install -g @angular/cli@17
```

#### Fetch dependencies
```bash
npm install
```

### Configuration

Update backend URL in these files:

**File: `src/environments/environment.development.ts`**
```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:8080'
};
```

**File: `src/environments/environment.ts`**
```typescript
export const environment = {
  production: true,
  apiUrl: 'http://localhost:8080'  // Change to your production backend URL
};
```

### Launch dev server
```bash
ng serve

# Or with auto-open browser
ng serve --open

# Or use npm script
npm start
```

Navigate to `http://localhost:4200/`. The application will automatically reload if you change any of the source files.

### Build
```bash
ng build
```

The build artifacts will be stored in the `dist/` directory.

### Build for Production
```bash
ng build --configuration production
```

### API Authentication

The frontend automatically:
1. Sends JWT token in `Authorization: Bearer <token>` header
2. Handles token storage after login
3. Validates token expiration
4. Redirects to login if token is invalid

#### Authentication Flow
1. User registers with email/password
2. Backend returns JWT token
3. Token is stored in browser storage
4. Token is sent with every API request
5. Token is validated on each request
6. Expired token → auto-redirect to login

### Available Routes
- `/` - Home page (public)
- `/login` - Login page
- `/register` - Registration page
- `/tenant/listings` - Browse listings (authenticated)
- `/tenant/search` - Search listings (authenticated)
- `/tenant/bookings` - View bookings (authenticated)
- `/landlord/properties` - Landlord properties (authenticated)
- `/landlord/reservations` - Landlord reservations (authenticated)

### Running Both Frontend and Backend

**Option 1: Two Terminal Windows**

Terminal 1 - Backend:
```bash
cd airbnb-clone-backend
./mvnw spring-boot:run
```

Terminal 2 - Frontend:
```bash
cd airbnb-clone-frontend
npm install  # First time only
ng serve --open
```

**Option 2: Using npm concurrently**

From root folder:
```bash
npm run dev  # If script is configured
```

### Documentation
- See `../airbnb-clone-backend/SETUP_AND_RUN_INSTRUCTIONS.md` for complete full-stack setup
- See `../airbnb-clone-backend/JWT_TESTING_GUIDE.md` for API testing examples
- See `../airbnb-clone-backend/JWT_MIGRATION_GUIDE.md` for technical details

