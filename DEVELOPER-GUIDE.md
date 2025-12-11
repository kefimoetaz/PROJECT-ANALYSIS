# K3FII Enterprise - Complete Developer Guide

## 📚 Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Setup Instructions](#setup-instructions)
4. [Code Structure](#code-structure)
5. [Key Concepts](#key-concepts)
6. [API Documentation](#api-documentation)
7. [Testing](#testing)
8. [Deployment](#deployment)

---

## 🎯 Project Overview

**K3FII Enterprise** is a full-stack User & Permissions Management System built with microservices architecture.

### Tech Stack
- **Frontend**: React 18 + TypeScript + Tailwind CSS
- **Backend**: Node.js + Express + TypeScript
- **Database**: MongoDB
- **Authentication**: JWT (JSON Web Tokens)
- **Containerization**: Docker + Docker Compose

### Key Features
- User Management (CRUD)
- Role-Based Access Control (RBAC)
- JWT Authentication
- Activity Logging
- Real-time Dashboard
- Microservices Architecture

---

## 🏗️ Architecture

### Microservices Structure
```
┌─────────────────┐
│  React Frontend │ :5173
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   API Gateway   │ :3000
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌─────────┐ ┌─────────┐
│  Auth   │ │  CRUD   │
│ Service │ │ Service │
│  :4001  │ │  :4002  │
└────┬────┘ └────┬────┘
     │           │
     └─────┬─────┘
           ▼
    ┌─────────────┐
    │   MongoDB   │ :27017
    └─────────────┘
```

### Service Responsibilities

**1. Frontend (Port 5173)**
- User interface
- State management
- API calls
- Routing

**2. API Gateway (Port 3000)**
- Single entry point
- Request routing
- CORS handling
- Rate limiting

**3. Auth Service (Port 4001)**
- User authentication
- JWT token generation
- Password hashing
- Token validation

**4. CRUD Service (Port 4002)**
- User CRUD operations
- Role management
- Permission management
- Activity logging

**5. MongoDB (Port 27017)**
- Data persistence
- User storage
- Role/Permission storage
- Activity logs

---

## 🚀 Setup Instructions

### Prerequisites
```bash
# Required software
- Node.js 18.x or higher
- Docker Desktop
- Git
- VS Code (recommended)
```

### Step 1: Clone the Project
```bash
git clone <repository-url>
cd k3fii-enterprise
```

### Step 2: Install Dependencies

**Auth Service:**
```bash
cd services/auth
npm install
```

**CRUD Service:**
```bash
cd services/crud
npm install
```

**Gateway:**
```bash
cd services/gateway
npm install
```

**Frontend:**
```bash
cd frontend
npm install
```

### Step 3: Environment Variables

**Auth Service** (`services/auth/.env`):
```env
NODE_ENV=production
PORT=4001
MONGODB_URI=mongodb://admin:password@mongodb:27017/users-permissions-dashboard?authSource=admin
JWT_ACCESS_SECRET=your-super-secret-access-token-key
JWT_REFRESH_SECRET=your-super-secret-refresh-token-key
JWT_ACCESS_EXPIRES_IN=7d
JWT_REFRESH_EXPIRES_IN=30d
```

**CRUD Service** (`services/crud/.env`):
```env
NODE_ENV=production
PORT=4002
MONGODB_URI=mongodb://admin:password@mongodb:27017/users-permissions-dashboard?authSource=admin
JWT_ACCESS_SECRET=your-super-secret-access-token-key
```

**Gateway** (`services/gateway/.env`):
```env
NODE_ENV=production
PORT=3000
AUTH_SERVICE_URL=http://auth:4001
CRUD_SERVICE_URL=http://crud:4002
```

### Step 4: Start with Docker
```bash
# Start all services
docker-compose -f docker-compose.microservices.yml up -d

# Check status
docker ps

# View logs
docker-compose -f docker-compose.microservices.yml logs -f
```

### Step 5: Access the Application
- Frontend: http://localhost:5173
- Login: admin@k3fii.com / admin123

---

## 📁 Code Structure

### Frontend Structure
```
frontend/
├── src/
│   ├── components/          # Reusable UI components
│   │   ├── Layout.tsx      # Main layout wrapper
│   │   ├── Sidebar.tsx     # Navigation sidebar
│   │   ├── UserModal.tsx   # User create/edit modal
│   │   └── RoleModal.tsx   # Role create/edit modal
│   │
│   ├── pages/              # Page components
│   │   ├── LoginPage.tsx   # Login page
│   │   ├── DashboardPage.tsx
│   │   ├── UsersPage.tsx
│   │   └── RolesPage.tsx
│   │
│   ├── contexts/           # React contexts
│   │   └── AuthContext.tsx # Authentication context
│   │
│   ├── lib/                # Utilities
│   │   └── api.ts          # API client (Axios)
│   │
│   ├── types/              # TypeScript types
│   │   └── index.ts        # Type definitions
│   │
│   ├── App.tsx             # Main app component
│   └── main.tsx            # Entry point
│
├── public/                 # Static assets
├── Dockerfile             # Docker configuration
└── package.json           # Dependencies
```

### Backend Structure (Auth Service)
```
services/auth/
├── src/
│   ├── routes/
│   │   └── auth.ts         # Auth routes (login, logout, refresh)
│   │
│   ├── models/
│   │   ├── User.ts         # User model
│   │   ├── Role.ts         # Role model
│   │   └── Permission.ts   # Permission model
│   │
│   ├── middleware/
│   │   ├── auth.ts         # JWT verification
│   │   ├── validation.ts   # Input validation
│   │   └── errorHandler.ts # Error handling
│   │
│   ├── utils/
│   │   └── jwt.ts          # JWT utilities
│   │
│   ├── config/
│   │   └── database.ts     # MongoDB connection
│   │
│   └── server.ts           # Express server
│
├── Dockerfile
└── package.json
```

### Backend Structure (CRUD Service)
```
services/crud/
├── src/
│   ├── routes/
│   │   ├── users.ts        # User CRUD routes
│   │   ├── roles.ts        # Role CRUD routes
│   │   ├── permissions.ts  # Permission routes
│   │   └── activities.ts   # Activity routes
│   │
│   ├── models/
│   │   ├── User.ts
│   │   ├── Role.ts
│   │   ├── Permission.ts
│   │   └── Activity.ts
│   │
│   ├── middleware/
│   │   └── auth.ts         # JWT verification
│   │
│   ├── schemas/
│   │   └── user.ts         # Validation schemas
│   │
│   └── server.ts
│
├── Dockerfile
└── package.json
```

---

## 🔑 Key Concepts

### 1. JWT Authentication Flow

```typescript
// Login Process
1. User submits email + password
2. Auth service validates credentials
3. Generate access token (7 days) + refresh token (30 days)
4. Return tokens to client
5. Client stores tokens in localStorage
6. Client includes access token in Authorization header

// Token Structure
{
  userId: "123456",
  iat: 1234567890,  // Issued at
  exp: 1234567890   // Expires at
}
```

### 2. Role-Based Access Control (RBAC)

```typescript
// Permission Structure
{
  name: "user.create",
  description: "Create new users",
  resource: "user",
  action: "create"
}

// Role Structure
{
  name: "admin",
  description: "Full system administrator",
  permissions: [
    "user.create",
    "user.read",
    "user.update",
    "user.delete",
    // ... more permissions
  ]
}

// User Structure
{
  email: "admin@k3fii.com",
  password: "hashed_password",
  roles: ["admin"],
  isActive: true
}
```

### 3. Middleware Chain

```typescript
// Request Flow
Request → CORS → Rate Limit → JWT Verify → Permission Check → Route Handler

// Example
app.use('/api/users', 
  authenticate,              // Verify JWT
  hasPermission('user.read'), // Check permission
  getUsersHandler            // Execute logic
);
```

### 4. Database Models

**User Model:**
```typescript
{
  email: string,
  password: string,  // bcrypt hashed
  firstName: string,
  lastName: string,
  roles: ObjectId[],  // References to Role
  isActive: boolean,
  lastLogin: Date,
  refreshTokens: string[],
  createdAt: Date,
  updatedAt: Date
}
```

**Role Model:**
```typescript
{
  name: string,
  description: string,
  permissions: ObjectId[],  // References to Permission
  isActive: boolean,
  createdAt: Date,
  updatedAt: Date
}
```

---

## 📡 API Documentation

### Authentication Endpoints

**POST /api/auth/login**
```json
// Request
{
  "email": "admin@k3fii.com",
  "password": "admin123"
}

// Response
{
  "message": "Login successful",
  "user": {
    "_id": "123",
    "email": "admin@k3fii.com",
    "firstName": "System",
    "lastName": "Administrator",
    "roles": [...]
  },
  "accessToken": "eyJhbGc...",
  "refreshToken": "eyJhbGc..."
}
```

**POST /api/auth/refresh**
```json
// Request
{
  "refreshToken": "eyJhbGc..."
}

// Response
{
  "accessToken": "eyJhbGc..."
}
```

**POST /api/auth/logout**
```json
// Request (with Authorization header)
{
  "refreshToken": "eyJhbGc..."
}

// Response
{
  "message": "Logout successful"
}
```

### User Endpoints

**GET /api/users**
```
Query Parameters:
- page: number (default: 1)
- limit: number (default: 10)
- search: string
- role: string
- isActive: boolean

Response:
{
  "users": [...],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 50,
    "pages": 5
  }
}
```

**POST /api/users**
```json
// Request
{
  "email": "user@example.com",
  "password": "password123",
  "firstName": "John",
  "lastName": "Doe",
  "roles": ["role_id_1", "role_id_2"]
}

// Response
{
  "message": "User created successfully",
  "user": {...}
}
```

**PUT /api/users/:id**
```json
// Request
{
  "firstName": "Jane",
  "lastName": "Smith",
  "roles": ["role_id_1"],
  "isActive": true
}

// Response
{
  "message": "User updated successfully",
  "user": {...}
}
```

**DELETE /api/users/:id**
```json
// Response
{
  "message": "User deleted successfully"
}
```

### Role Endpoints

**GET /api/roles**
```json
// Response
{
  "roles": [
    {
      "_id": "123",
      "name": "admin",
      "description": "Full administrator",
      "permissions": [...],
      "isActive": true
    }
  ]
}
```

**POST /api/roles**
```json
// Request
{
  "name": "manager",
  "description": "Department manager",
  "permissions": ["perm_id_1", "perm_id_2"]
}

// Response
{
  "message": "Role created successfully",
  "role": {...}
}
```

---

## 🧪 Testing

### Manual Testing

**1. Test Login:**
```powershell
.\test-auth-direct.ps1
```

**2. Test All Services:**
```powershell
.\final-test.ps1
```

**3. Test Microservices:**
```powershell
.\test-microservices.ps1
```

### API Testing with Postman

1. Import `postman-collection.json`
2. Set environment variables:
   - `base_url`: http://localhost:3000
   - `access_token`: (will be set automatically after login)
3. Run collection tests

### Browser Testing Checklist

- [ ] Login with valid credentials
- [ ] Login with invalid credentials (should fail)
- [ ] View dashboard
- [ ] Create new user
- [ ] Edit user
- [ ] Delete user
- [ ] Create new role
- [ ] Assign permissions to role
- [ ] View activity logs
- [ ] Logout

---

## 🚢 Deployment

### Docker Deployment (Production)

**1. Build all services:**
```bash
docker-compose -f docker-compose.microservices.yml build
```

**2. Start services:**
```bash
docker-compose -f docker-compose.microservices.yml up -d
```

**3. Check health:**
```bash
docker ps
docker-compose -f docker-compose.microservices.yml logs
```

### Environment Variables for Production

Update these in `docker-compose.microservices.yml`:
- Change JWT secrets to strong random strings
- Update MongoDB credentials
- Set NODE_ENV=production
- Configure CORS origins

### Scaling Services

```bash
# Scale CRUD service to 3 instances
docker-compose -f docker-compose.microservices.yml up -d --scale crud=3

# Scale Auth service to 2 instances
docker-compose -f docker-compose.microservices.yml up -d --scale auth=2
```

---

## 🔧 Common Issues & Solutions

### Issue 1: Port Already in Use
```bash
# Find process using port
netstat -ano | findstr :3000

# Kill process
taskkill /PID <process_id> /F
```

### Issue 2: Docker Not Starting
```bash
# Restart Docker Desktop
# Then:
docker-compose -f docker-compose.microservices.yml down
docker-compose -f docker-compose.microservices.yml up -d
```

### Issue 3: Database Connection Failed
```bash
# Check MongoDB is running
docker ps | findstr mongodb

# Restart MongoDB
docker restart users-permissions-mongodb
```

### Issue 4: Frontend Not Loading
```bash
# Rebuild frontend
docker-compose -f docker-compose.microservices.yml build frontend
docker-compose -f docker-compose.microservices.yml up -d frontend
```

---

## 📚 Learning Resources

### Microservices
- [Microservices Architecture](https://microservices.io/)
- [API Gateway Pattern](https://microservices.io/patterns/apigateway.html)

### React & TypeScript
- [React Documentation](https://react.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

### Node.js & Express
- [Express.js Guide](https://expressjs.com/en/guide/routing.html)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

### MongoDB
- [MongoDB University](https://university.mongodb.com/)
- [Mongoose Documentation](https://mongoosejs.com/docs/)

### Docker
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)

---

## 🎯 Next Steps for Your Friend

1. **Understand the Architecture**
   - Study the microservices diagram
   - Understand service communication
   - Learn about API Gateway pattern

2. **Explore the Code**
   - Start with frontend (React components)
   - Then backend services (Express routes)
   - Study the database models

3. **Run the Application**
   - Follow setup instructions
   - Test all features
   - Check logs and debug

4. **Modify and Experiment**
   - Add a new permission
   - Create a new role
   - Add a new API endpoint
   - Customize the UI

5. **Build Similar Projects**
   - Start with a simpler version
   - Add features incrementally
   - Follow the same architecture pattern

---

## 📞 Support

If you need help:
1. Check the logs: `docker-compose logs -f`
2. Review this guide
3. Check README.md for quick reference
4. Test with provided scripts

---

**Good luck building your application! 🚀**

*This guide covers everything needed to understand, run, and build upon this project.*
