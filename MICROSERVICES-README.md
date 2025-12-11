# 🏗️ Microservices Architecture - Users & Permissions Dashboard

This project has been converted from a monolithic architecture to a microservices architecture with 5 independent services.

## 📐 Architecture Overview

```
┌─────────────┐
│   Frontend  │ (React + Vite)
│   :5173     │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ API Gateway │ (Express Proxy)
│   :3000     │
└──────┬──────┘
       │
       ├──────────────┬──────────────┐
       ▼              ▼              ▼
┌──────────┐   ┌──────────┐   ┌──────────┐
│   Auth   │   │   CRUD   │   │ MongoDB  │
│  :4001   │   │  :4002   │   │  :27017  │
└──────────┘   └──────────┘   └──────────┘
```

## 🎯 Services

### 1. Auth Service (Port 4001)
Handles all authentication operations:
- `POST /auth/login` - User login
- `POST /auth/refresh` - Refresh access token
- `POST /auth/logout` - User logout
- `GET /auth/me` - Get current user profile

### 2. CRUD Service (Port 4002)
Manages all CRUD operations with JWT validation:
- `/users` - User management (CRUD)
- `/roles` - Role management (CRUD)
- `/permissions` - Permission listing
- `/activities` - Activity logs

### 3. API Gateway (Port 3000)
Routes requests to appropriate services:
- `/api/auth/*` → Auth Service
- `/api/users/*` → CRUD Service
- `/api/roles/*` → CRUD Service
- `/api/permissions/*` → CRUD Service
- `/api/activities/*` → CRUD Service

### 4. Frontend (Port 5173)
React dashboard that communicates with API Gateway

### 5. MongoDB (Port 27017)
Shared database for all services

## 🚀 Quick Start

### Prerequisites
- Docker & Docker Compose
- Node.js 18+ (for local development)

### Running with Docker

1. **Build and start all services:**
```bash
docker-compose -f docker-compose.microservices.yml up --build
```

2. **Seed the database (first time only):**
```bash
# Connect to the backend container (if you have the old one) or run locally
cd backend
npm run seed
```

3. **Access the application:**
- Frontend: http://localhost:5173
- API Gateway: http://localhost:3000
- Auth Service: http://localhost:4001
- CRUD Service: http://localhost:4002

### Running Locally (Development)

1. **Start MongoDB:**
```bash
docker run -d -p 27017:27017 --name mongodb \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  mongo:7
```

2. **Start Auth Service:**
```bash
cd services/auth
npm install
cp .env.example .env
npm run dev
```

3. **Start CRUD Service:**
```bash
cd services/crud
npm install
cp .env.example .env
npm run dev
```

4. **Start API Gateway:**
```bash
cd services/gateway
npm install
cp .env.example .env
npm run dev
```

5. **Start Frontend:**
```bash
cd frontend
npm install
npm run dev
```

## 🔐 Environment Variables

### Auth Service (.env)
```env
NODE_ENV=production
PORT=4001
MONGODB_URI=mongodb://admin:password@mongodb:27017/users-permissions-dashboard?authSource=admin
JWT_ACCESS_SECRET=your-super-secret-access-token-key-here
JWT_REFRESH_SECRET=your-super-secret-refresh-token-key-here
JWT_ACCESS_EXPIRES_IN=7d
JWT_REFRESH_EXPIRES_IN=30d
```

### CRUD Service (.env)
```env
NODE_ENV=production
PORT=4002
MONGODB_URI=mongodb://admin:password@mongodb:27017/users-permissions-dashboard?authSource=admin
JWT_ACCESS_SECRET=your-super-secret-access-token-key-here
```

### API Gateway (.env)
```env
NODE_ENV=production
PORT=3000
AUTH_SERVICE_URL=http://auth:4001
CRUD_SERVICE_URL=http://crud:4002
```

## 📁 Project Structure

```
project-root/
├── services/
│   ├── auth/                    # Authentication service
│   │   ├── src/
│   │   │   ├── config/         # Database config
│   │   │   ├── middleware/     # Auth & validation
│   │   │   ├── models/         # Mongoose models
│   │   │   ├── routes/         # Auth routes
│   │   │   ├── schemas/        # Zod schemas
│   │   │   ├── services/       # Activity logging
│   │   │   ├── utils/          # JWT utilities
│   │   │   └── server.ts       # Entry point
│   │   ├── Dockerfile
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── crud/                    # CRUD operations service
│   │   ├── src/
│   │   │   ├── config/         # Database config
│   │   │   ├── middleware/     # Auth & validation
│   │   │   ├── models/         # Mongoose models
│   │   │   ├── routes/         # CRUD routes
│   │   │   ├── schemas/        # Zod schemas
│   │   │   └── server.ts       # Entry point
│   │   ├── Dockerfile
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── gateway/                 # API Gateway
│       ├── src/
│       │   └── server.ts       # Proxy configuration
│       ├── Dockerfile
│       ├── package.json
│       └── tsconfig.json
│
├── frontend/                    # React frontend
│   ├── src/
│   ├── Dockerfile
│   └── package.json
│
├── docker-compose.microservices.yml
└── MICROSERVICES-README.md
```

## 🔄 Migration from Monolith

### What Changed:

1. **Backend Split:**
   - Auth routes → Auth Service
   - CRUD routes → CRUD Service
   - API Gateway routes requests

2. **JWT Validation:**
   - Auth Service: Issues tokens
   - CRUD Service: Validates tokens independently
   - Both services share the same JWT secret

3. **Database:**
   - Shared MongoDB instance
   - All services connect to the same database

4. **Frontend:**
   - No changes needed
   - Still calls `/api/*` endpoints
   - Gateway handles routing

## 🛡️ Security Features

- JWT-based authentication
- Role-Based Access Control (RBAC)
- Permission validation on CRUD operations
- Rate limiting on API Gateway
- Helmet security headers
- CORS configuration
- Input validation with Zod

## 📊 Health Checks

Each service exposes a `/health` endpoint:

```bash
# Check all services
curl http://localhost:3000/health  # Gateway
curl http://localhost:4001/health  # Auth
curl http://localhost:4002/health  # CRUD
```

## 🧪 Testing

### Test Auth Service:
```bash
# Login
curl -X POST http://localhost:4001/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"Admin@1234"}'

# Get current user
curl http://localhost:4001/auth/me \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Test CRUD Service:
```bash
# Get users (requires valid JWT)
curl http://localhost:4002/users \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Test via Gateway:
```bash
# Login via gateway
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"Admin@1234"}'

# Get users via gateway
curl http://localhost:3000/api/users \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## 🐛 Troubleshooting

### Service won't start:
```bash
# Check logs
docker-compose -f docker-compose.microservices.yml logs auth
docker-compose -f docker-compose.microservices.yml logs crud
docker-compose -f docker-compose.microservices.yml logs gateway
```

### Database connection issues:
```bash
# Restart MongoDB
docker-compose -f docker-compose.microservices.yml restart mongodb

# Check MongoDB logs
docker-compose -f docker-compose.microservices.yml logs mongodb
```

### Port conflicts:
```bash
# Check what's using the ports
netstat -ano | findstr :3000
netstat -ano | findstr :4001
netstat -ano | findstr :4002
```

## 🔧 Development Tips

1. **Hot Reload:** Use `npm run dev` in each service for development
2. **Debugging:** Check individual service logs for errors
3. **Database:** Use MongoDB Compass to inspect data
4. **API Testing:** Use Postman or the provided test scripts

## 📈 Scaling

Each service can be scaled independently:

```bash
# Scale CRUD service to 3 instances
docker-compose -f docker-compose.microservices.yml up --scale crud=3
```

## 🎯 Production Deployment

1. **Update environment variables** with production values
2. **Use secrets management** for sensitive data
3. **Add load balancer** in front of API Gateway
4. **Enable HTTPS** with SSL certificates
5. **Set up monitoring** and logging
6. **Configure backup** for MongoDB

## 📝 Default Credentials

| Role | Email | Password |
|------|-------|----------|
| Admin | admin@example.com | Admin@1234 |
| Manager | manager@example.com | Manager@1234 |
| User | user1@example.com | User@1234 |

## 🤝 Contributing

When adding new features:
1. Determine which service handles the feature
2. Add routes to the appropriate service
3. Update API Gateway if needed
4. Update this README

## 📄 License

MIT License - See LICENSE file for details
