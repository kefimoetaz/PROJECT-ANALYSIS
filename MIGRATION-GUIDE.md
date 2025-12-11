# 🔄 Migration Guide: Monolith to Microservices

## Overview

This guide explains how the monolithic application was converted to microservices and how to use the new architecture.

## Architecture Comparison

### Before (Monolith)
```
Frontend → Backend (Port 3000) → MongoDB
           ├── /api/auth/*
           ├── /api/users/*
           ├── /api/roles/*
           └── /api/permissions/*
```

### After (Microservices)
```
Frontend → Gateway (Port 3000) → Auth Service (Port 4001) → MongoDB
                                → CRUD Service (Port 4002) → MongoDB
```

## What Was Split

### Auth Service (services/auth/)
**Responsibilities:**
- User authentication (login/logout)
- JWT token generation
- Token refresh
- Current user profile

**Endpoints:**
- `POST /auth/login`
- `POST /auth/refresh`
- `POST /auth/logout`
- `GET /auth/me`

**Files Moved:**
- `routes/auth.ts` → `services/auth/src/routes/auth.ts`
- `utils/jwt.ts` → `services/auth/src/utils/jwt.ts`
- `middleware/auth.ts` → `services/auth/src/middleware/auth.ts`
- `models/*` → `services/auth/src/models/*`

### CRUD Service (services/crud/)
**Responsibilities:**
- User management (CRUD)
- Role management (CRUD)
- Permission listing
- Activity logs
- JWT validation for all operations

**Endpoints:**
- `/users/*` - All user operations
- `/roles/*` - All role operations
- `/permissions/*` - Permission listing
- `/activities/*` - Activity logs

**Files Moved:**
- `routes/users.ts` → `services/crud/src/routes/users.ts`
- `routes/roles.ts` → `services/crud/src/routes/roles.ts`
- `routes/permissions.ts` → `services/crud/src/routes/permissions.ts`
- `routes/activities.ts` → `services/crud/src/routes/activities.ts`
- `models/*` → `services/crud/src/models/*`
- `middleware/auth.ts` → `services/crud/src/middleware/auth.ts`

### API Gateway (services/gateway/)
**Responsibilities:**
- Route requests to appropriate services
- Rate limiting
- CORS handling
- Security headers

**Routing:**
- `/api/auth/*` → Auth Service
- `/api/users/*` → CRUD Service
- `/api/roles/*` → CRUD Service
- `/api/permissions/*` → CRUD Service
- `/api/activities/*` → CRUD Service

## Key Changes

### 1. JWT Validation
**Before:** Centralized in backend
**After:** Each service validates JWT independently

Both services share the same `JWT_ACCESS_SECRET` to validate tokens.

### 2. Database Access
**Before:** Single backend connects to MongoDB
**After:** Both Auth and CRUD services connect to the same MongoDB instance

### 3. Frontend Communication
**Before:** Direct to backend at `http://localhost:3000`
**After:** Through API Gateway at `http://localhost:3000` (no change needed!)

### 4. Error Handling
Each service has its own error handler but maintains consistent error responses.

### 5. RBAC (Role-Based Access Control)
**Before:** Centralized in backend middleware
**After:** CRUD service validates permissions using JWT user data

## Running the New Architecture

### Option 1: Docker Compose (Recommended)
```bash
# Windows
.\start-microservices.ps1

# Linux/Mac
./start-microservices.sh

# Or manually
docker-compose -f docker-compose.microservices.yml up --build
```

### Option 2: Local Development
```bash
# Terminal 1 - MongoDB
docker run -d -p 27017:27017 --name mongodb \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  mongo:7

# Terminal 2 - Auth Service
cd services/auth
npm install
cp .env.example .env
npm run dev

# Terminal 3 - CRUD Service
cd services/crud
npm install
cp .env.example .env
npm run dev

# Terminal 4 - API Gateway
cd services/gateway
npm install
cp .env.example .env
npm run dev

# Terminal 5 - Frontend
cd frontend
npm install
npm run dev
```

## Testing the Migration

### 1. Test Auth Service Directly
```bash
# Login
curl -X POST http://localhost:4001/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"Admin@1234"}'

# Response includes accessToken
```

### 2. Test CRUD Service Directly
```bash
# Get users (use token from login)
curl http://localhost:4002/users \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### 3. Test via API Gateway
```bash
# Login via gateway
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"Admin@1234"}'

# Get users via gateway
curl http://localhost:3000/api/users \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### 4. Test Frontend
Open http://localhost:5173 and login normally. Everything should work as before!

## Database Seeding

The database structure hasn't changed, so you can use the existing seed script:

```bash
# If you have the old backend
cd backend
npm run seed

# Or create a new seed script in one of the services
cd services/auth
npm run seed  # (if you add the seed script)
```

## Environment Variables

### Critical: JWT Secrets Must Match!

Both Auth and CRUD services must use the **same** `JWT_ACCESS_SECRET`:

**services/auth/.env:**
```env
JWT_ACCESS_SECRET=your-super-secret-access-token-key-here
JWT_REFRESH_SECRET=your-super-secret-refresh-token-key-here
```

**services/crud/.env:**
```env
JWT_ACCESS_SECRET=your-super-secret-access-token-key-here
```

If these don't match, CRUD service won't be able to validate tokens from Auth service!

## Common Issues

### Issue 1: "Invalid token" in CRUD service
**Cause:** JWT secrets don't match between services
**Solution:** Ensure both services use the same `JWT_ACCESS_SECRET`

### Issue 2: Services can't connect to MongoDB
**Cause:** MongoDB not ready or wrong connection string
**Solution:** 
- Check MongoDB is running: `docker ps | grep mongodb`
- Verify connection string in .env files
- Wait for MongoDB to be fully ready (healthcheck)

### Issue 3: Gateway returns 503
**Cause:** Backend services not running
**Solution:** Check service health:
```bash
curl http://localhost:4001/health
curl http://localhost:4002/health
```

### Issue 4: CORS errors in frontend
**Cause:** Gateway CORS configuration
**Solution:** Check gateway allows frontend origin in `services/gateway/src/server.ts`

## Rollback to Monolith

If you need to rollback:

```bash
# Stop microservices
docker-compose -f docker-compose.microservices.yml down

# Start original monolith
docker-compose up --build
```

The original monolithic setup is still available in:
- `backend/` folder
- `docker-compose.yml` (original file)

## Benefits of Microservices

1. **Independent Scaling:** Scale Auth and CRUD services separately
2. **Independent Deployment:** Deploy services without affecting others
3. **Technology Flexibility:** Each service can use different tech stack
4. **Fault Isolation:** If one service fails, others continue working
5. **Team Organization:** Different teams can own different services

## Monitoring

Check service health:
```bash
# All services
curl http://localhost:3000/health  # Gateway
curl http://localhost:4001/health  # Auth
curl http://localhost:4002/health  # CRUD

# View logs
docker-compose -f docker-compose.microservices.yml logs -f auth
docker-compose -f docker-compose.microservices.yml logs -f crud
docker-compose -f docker-compose.microservices.yml logs -f gateway
```

## Next Steps

1. ✅ Test all endpoints through the gateway
2. ✅ Verify RBAC permissions work correctly
3. ✅ Test user creation, update, and deletion
4. ✅ Test role and permission management
5. ✅ Check activity logging
6. 🔄 Add monitoring and logging (Prometheus, Grafana)
7. 🔄 Add service discovery (Consul, Eureka)
8. 🔄 Add API documentation (Swagger per service)
9. 🔄 Add integration tests
10. 🔄 Set up CI/CD pipeline

## Support

If you encounter issues:
1. Check service logs
2. Verify environment variables
3. Ensure all services are healthy
4. Check network connectivity between services
5. Review this migration guide

Happy microservicing! 🚀
