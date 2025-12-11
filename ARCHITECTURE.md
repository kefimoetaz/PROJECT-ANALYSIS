# 🏗️ Microservices Architecture Documentation

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                             │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  React Frontend (Port 5173)                               │  │
│  │  - Dashboard UI                                           │  │
│  │  - User Management                                        │  │
│  │  - Role & Permission Management                           │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────────────────────────┬─────────────────────────────────────┘
                            │ HTTP/REST
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API GATEWAY LAYER                           │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  API Gateway (Port 3000)                                  │  │
│  │  - Request Routing                                        │  │
│  │  - Rate Limiting                                          │  │
│  │  - CORS Handling                                          │  │
│  │  - Security Headers                                       │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────────────┬───────────────────────┬─────────────────────────┘
                │                       │
    ┌───────────┴──────────┐   ┌───────┴──────────┐
    │ /api/auth/*          │   │ /api/users/*     │
    │ /api/me              │   │ /api/roles/*     │
    │                      │   │ /api/permissions/*│
    │                      │   │ /api/activities/* │
    ▼                      │   ▼                   │
┌─────────────────────┐   │   ┌─────────────────────┐
│  SERVICE LAYER      │   │   │  SERVICE LAYER      │
│                     │   │   │                     │
│  Auth Service       │   │   │  CRUD Service       │
│  (Port 4001)        │   │   │  (Port 4002)        │
│                     │   │   │                     │
│  - Login            │   │   │  - User CRUD        │
│  - Logout           │   │   │  - Role CRUD        │
│  - Token Refresh    │   │   │  - Permission List  │
│  - Current User     │   │   │  - Activity Logs    │
│  - JWT Generation   │   │   │  - JWT Validation   │
└──────────┬──────────┘   │   └──────────┬──────────┘
           │              │              │
           │              │              │
           └──────────────┴──────────────┘
                          │
                          ▼
           ┌──────────────────────────────┐
           │     DATA LAYER               │
           │                              │
           │  MongoDB (Port 27017)        │
           │  - Users Collection          │
           │  - Roles Collection          │
           │  - Permissions Collection    │
           │  - Activities Collection     │
           └──────────────────────────────┘
```

## Service Responsibilities

### 1. Frontend Service
**Technology:** React 18 + TypeScript + Vite + Tailwind CSS

**Responsibilities:**
- User interface rendering
- Client-side routing
- Form validation
- State management
- API communication via Gateway

**Key Features:**
- Dashboard with analytics
- User management interface
- Role and permission management
- Real-time notifications
- Responsive design

### 2. API Gateway
**Technology:** Express.js + http-proxy-middleware

**Responsibilities:**
- Route incoming requests to appropriate services
- Apply rate limiting
- Handle CORS
- Add security headers
- Load balancing (future)

**Routing Rules:**
```
/api/auth/*        → Auth Service (4001)
/api/users/*       → CRUD Service (4002)
/api/roles/*       → CRUD Service (4002)
/api/permissions/* → CRUD Service (4002)
/api/activities/*  → CRUD Service (4002)
```

**Security:**
- Helmet.js for security headers
- Rate limiting (100 requests per 15 minutes)
- CORS configuration
- Request/response logging

### 3. Auth Service
**Technology:** Express.js + TypeScript + MongoDB + JWT

**Responsibilities:**
- User authentication
- JWT token generation and validation
- Token refresh mechanism
- Session management
- Activity logging for auth events

**Endpoints:**
```
POST   /auth/login     - Authenticate user
POST   /auth/refresh   - Refresh access token
POST   /auth/logout    - Invalidate refresh token
GET    /auth/me        - Get current user profile
```

**Security:**
- bcrypt password hashing (12 rounds)
- JWT with configurable expiration
- Refresh token rotation
- Account lockout (future)

**Database Models:**
- User
- Role
- Permission
- Activity

### 4. CRUD Service
**Technology:** Express.js + TypeScript + MongoDB

**Responsibilities:**
- User management (CRUD operations)
- Role management (CRUD operations)
- Permission listing
- Activity log retrieval
- JWT validation for all operations
- RBAC enforcement

**Endpoints:**
```
# Users
GET    /users           - List users (paginated)
POST   /users           - Create user
GET    /users/:id       - Get user by ID
PUT    /users/:id       - Update user
PATCH  /users/:id/status - Update user status
DELETE /users/:id       - Delete user

# Roles
GET    /roles           - List all roles
POST   /roles           - Create role
GET    /roles/:id       - Get role by ID
PUT    /roles/:id       - Update role
POST   /roles/:id/permissions - Attach permissions
DELETE /roles/:id/permissions - Detach permissions
DELETE /roles/:id       - Delete role

# Permissions
GET    /permissions     - List all permissions

# Activities
GET    /activities      - List activity logs
```

**Security:**
- JWT validation on all endpoints
- Permission-based access control
- Input validation with Zod
- SQL injection prevention (MongoDB)

**Database Models:**
- User
- Role
- Permission
- Activity

### 5. MongoDB Database
**Technology:** MongoDB 7

**Responsibilities:**
- Persistent data storage
- Data relationships
- Indexing for performance
- Backup and recovery

**Collections:**
```
users
  - _id, email, password, firstName, lastName
  - roles (ref), isActive, lastLogin, refreshTokens
  - createdAt, updatedAt

roles
  - _id, name, description
  - permissions (ref), isActive
  - createdAt, updatedAt

permissions
  - _id, name, description, resource, action
  - createdAt, updatedAt

activities
  - _id, type, description
  - userId (ref), targetUserId (ref), targetRoleId (ref)
  - metadata, ipAddress, userAgent
  - createdAt, updatedAt
```

## Data Flow

### Authentication Flow
```
1. User enters credentials in Frontend
2. Frontend → Gateway: POST /api/auth/login
3. Gateway → Auth Service: POST /auth/login
4. Auth Service validates credentials with MongoDB
5. Auth Service generates JWT tokens
6. Auth Service logs activity
7. Auth Service → Gateway: { user, accessToken, refreshToken }
8. Gateway → Frontend: { user, accessToken, refreshToken }
9. Frontend stores tokens in memory/localStorage
```

### CRUD Operation Flow
```
1. User performs action in Frontend (e.g., create user)
2. Frontend → Gateway: POST /api/users (with JWT in header)
3. Gateway → CRUD Service: POST /users (with JWT)
4. CRUD Service validates JWT
5. CRUD Service checks user permissions
6. CRUD Service performs operation in MongoDB
7. CRUD Service → Gateway: { success, data }
8. Gateway → Frontend: { success, data }
9. Frontend updates UI
```

### Token Refresh Flow
```
1. Frontend detects expired access token
2. Frontend → Gateway: POST /api/auth/refresh (with refreshToken)
3. Gateway → Auth Service: POST /auth/refresh
4. Auth Service validates refresh token
5. Auth Service generates new access token
6. Auth Service → Gateway: { accessToken }
7. Gateway → Frontend: { accessToken }
8. Frontend retries original request with new token
```

## Security Architecture

### Authentication & Authorization
```
┌─────────────────────────────────────────────────────────┐
│  Request with JWT Token                                  │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  API Gateway                                             │
│  - Rate Limiting                                         │
│  - CORS Validation                                       │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  Service (Auth or CRUD)                                  │
│  1. Extract JWT from Authorization header                │
│  2. Verify JWT signature                                 │
│  3. Check token expiration                               │
│  4. Load user from database                              │
│  5. Check user is active                                 │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  RBAC Middleware (CRUD Service only)                     │
│  1. Load user roles                                      │
│  2. Load role permissions                                │
│  3. Check required permission                            │
│  4. Allow/Deny request                                   │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  Route Handler                                           │
│  - Execute business logic                                │
│  - Return response                                       │
└─────────────────────────────────────────────────────────┘
```

### JWT Token Structure
```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "userId": "507f1f77bcf86cd799439011",
    "iat": 1699564800,
    "exp": 1700169600
  },
  "signature": "..."
}
```

## Communication Patterns

### Synchronous Communication
All services use HTTP/REST for synchronous communication:
- Frontend ↔ Gateway: REST API
- Gateway ↔ Services: HTTP Proxy
- Services ↔ MongoDB: MongoDB Driver

### Error Handling
```
Service Error → Gateway → Frontend
     ↓
  Log Error
     ↓
Return Standardized Error Response:
{
  "error": "Error message",
  "details": [...],  // Optional
  "stack": "..."     // Development only
}
```

## Scalability

### Horizontal Scaling
Each service can be scaled independently:

```bash
# Scale CRUD service to 3 instances
docker-compose -f docker-compose.microservices.yml up --scale crud=3

# Scale Auth service to 2 instances
docker-compose -f docker-compose.microservices.yml up --scale auth=2
```

### Load Balancing
API Gateway can distribute requests across multiple service instances.

### Database Scaling
- Read replicas for read-heavy operations
- Sharding for large datasets
- Connection pooling

## Monitoring & Observability

### Health Checks
Each service exposes `/health` endpoint:
```json
{
  "service": "auth",
  "status": "OK",
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

### Logging
- Structured logging with timestamps
- Request/response logging
- Error logging with stack traces
- Activity logging for audit trail

### Metrics (Future)
- Request count per endpoint
- Response time percentiles
- Error rates
- Active connections

## Deployment

### Docker Compose (Development)
```bash
docker-compose -f docker-compose.microservices.yml up
```

### Kubernetes (Production)
```yaml
# Example deployment structure
- Namespace: users-permissions
- Deployments: auth, crud, gateway, frontend
- Services: auth-svc, crud-svc, gateway-svc
- Ingress: Route external traffic
- ConfigMaps: Environment variables
- Secrets: JWT secrets, DB credentials
```

## Performance Considerations

### Caching Strategy
- Frontend: Cache user data in memory
- Gateway: Cache service discovery
- Services: Cache user permissions
- Database: Index frequently queried fields

### Database Indexes
```javascript
// Users
users.createIndex({ email: 1 })
users.createIndex({ firstName: 1, lastName: 1 })

// Roles
roles.createIndex({ name: 1 })

// Permissions
permissions.createIndex({ resource: 1, action: 1 })

// Activities
activities.createIndex({ createdAt: -1 })
activities.createIndex({ type: 1, createdAt: -1 })
```

### Connection Pooling
- MongoDB connection pool: 10 connections per service
- HTTP keep-alive for service-to-service communication

## Future Enhancements

1. **Service Discovery:** Consul or Eureka
2. **API Documentation:** Swagger per service
3. **Message Queue:** RabbitMQ or Kafka for async operations
4. **Caching Layer:** Redis for session and data caching
5. **Monitoring:** Prometheus + Grafana
6. **Tracing:** Jaeger or Zipkin
7. **Circuit Breaker:** Resilience4j
8. **API Versioning:** /v1/, /v2/ endpoints
9. **GraphQL Gateway:** Alternative to REST
10. **WebSocket Support:** Real-time notifications

## Conclusion

This microservices architecture provides:
- ✅ Independent service deployment
- ✅ Horizontal scalability
- ✅ Fault isolation
- ✅ Technology flexibility
- ✅ Team autonomy
- ✅ Maintainability
- ✅ Production-ready security

The architecture maintains all features from the monolith while providing better scalability and maintainability.
