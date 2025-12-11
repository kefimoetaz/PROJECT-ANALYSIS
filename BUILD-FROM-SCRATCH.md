
## 🎯 What You'll Build
A complete User & Permissions Management System with microservices architecture.

**Time:** 2-3 days | **Difficulty:** Intermediate

---

## 📚 Learning Path

### Day 1: Backend Foundation
1. Setup MongoDB
2. Build Auth Service (JWT authentication)
3. Build CRUD Service (User management)
4. Build API Gateway

### Day 2: Frontend
1. Setup React + TypeScript
2. Build authentication flow
3. Create dashboard
4. Build user management UI

### Day 3: Integration & Docker
1. Connect all services
2. Add role-based access control
3. Dockerize everything
4. Test and deploy

---

## 🛠️ Prerequisites

### Required Software
```
✅ Node.js 18+ (https://nodejs.org)
✅ Docker Desktop (https://docker.com)
✅ VS Code (https://code.visualstudio.com)
✅ Git (https://git-scm.com)
✅ Postman (for API testing)
```

### Required Knowledge
- JavaScript/TypeScript basics
- React fundamentals
- Basic understanding of REST APIs
- Basic Docker concepts

---

## 📁 Step 1: Project Structure (10 min)

### Create Folders
```bash
mkdir my-enterprise-app
cd my-enterprise-app

# Backend services
mkdir -p services/auth/src/{routes,models,middleware,utils,config}
mkdir -p services/crud/src/{routes,models,middleware,schemas}
mkdir -p services/gateway/src

# Frontend
mkdir -p frontend/src/{components,pages,contexts,lib,types}
```

### Initialize Git
```bash
git init
```

Create `.gitignore`:
```
node_modules/
dist/
.env
*.log
```

---

## 🗄️ Step 2: MongoDB Setup (15 min)

### Create docker-compose.yml
```yaml
version: '3.8'
services:
  mongodb:
    image: mongo:7
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
    volumes:
      - mongodb_data:/data/db
volumes:
  mongodb_data:
```

### Start MongoDB
```bash
docker-compose up -d
```

---

## 🔐 Step 3: Auth Service (3 hours)

### 3.1 Initialize Project
```bash
cd services/auth
npm init -y
```

### 3.2 Install Dependencies
```bash
npm install express mongoose bcryptjs jsonwebtoken cors helmet dotenv
npm install -D typescript @types/express @types/node @types/bcryptjs @types/jsonwebtoken ts-node nodemon
```

### 3.3 TypeScript Config
Create `tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  }
}
```

### 3.4 Package Scripts
Update `package.json`:
```json
"scripts": {
  "dev": "nodemon src/server.ts",
  "build": "tsc",
  "start": "node dist/server.ts"
}
```

---

## 📝 Step 4: User Model (30 min)

Create `services/auth/src/models/User.ts`:

```typescript
import mongoose, { Document, Schema } from 'mongoose';
import bcrypt from 'bcryptjs';

export interface IUser extends Document {
  email: string;
  password: string;
  firstName: string;
  lastName: string;
  roles: mongoose.Types.ObjectId[];
  isActive: boolean;
  refreshTokens: string[];
  comparePassword(password: string): Promise<boolean>;
}

const userSchema = new Schema<IUser>({
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  firstName: { type: String, required: true },
  lastName: { type: String, required: true },
  roles: [{ type: Schema.Types.ObjectId, ref: 'Role' }],
  isActive: { type: Boolean, default: true },
  refreshTokens: [String]
}, { timestamps: true });

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// Compare password method
userSchema.methods.comparePassword = async function(password: string) {
  return bcrypt.compare(password, this.password);
};

export const User = mongoose.model<IUser>('User', userSchema);
```

**What this does:**
- Defines User structure
- Auto-hashes passwords with bcrypt
- Provides password comparison method

---

## 🔑 Step 5: JWT Utilities (20 min)

Create `services/auth/src/utils/jwt.ts`:
```typescript
import jwt from 'jsonwebtoken';

export interface TokenPayload {
  userId: string;
}

const ACCESS_SECRET = process.env.JWT_ACCESS_SECRET || 'access-secret';
const REFRESH_SECRET = process.env.JWT_REFRESH_SECRET || 'refresh-secret';

export const generateAccessToken = (payload: TokenPayload): string => {
  return jwt.sign(payload, ACCESS_SECRET, { expiresIn: '7d' });
};

export const generateRefreshToken = (payload: TokenPayload): string => {
  return jwt.sign(payload, REFRESH_SECRET, { expiresIn: '30d' });
};

export const verifyAccessToken = (token: string): TokenPayload => {
  return jwt.verify(token, ACCESS_SECRET) as TokenPayload;
};

export const verifyRefreshToken = (token: string): TokenPayload => {
  return jwt.verify(token, REFRESH_SECRET) as TokenPayload;
};
```

**What this does:**
- Creates JWT tokens
- Verifies JWT tokens
- Access token: 7 days
- Refresh token: 30 days

---

## 🛣️ Step 6: Auth Routes (45 min)

Create `services/auth/src/routes/auth.ts`:
```typescript
import express from 'express';
import { User } from '../models/User';
import { generateAccessToken, generateRefreshToken } from '../utils/jwt';

const router = express.Router();

// Login
router.post('/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    
    const user = await User.findOne({ email });
    if (!user || !user.isActive) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    const isValid = await user.comparePassword(password);
    if (!isValid) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    const accessToken = generateAccessToken({ userId: user._id.toString() });
    const refreshToken = generateRefreshToken({ userId: user._id.toString() });
    
    user.refreshTokens.push(refreshToken);
    await user.save();
    
    res.json({
      message: 'Login successful',
      user: {
        _id: user._id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName
      },
      accessToken,
      refreshToken
    });
  } catch (error) {
    res.status(500).json({ error: 'Server error' });
  }
});

export default router;
```

**What this does:**
- Validates user credentials
- Generates JWT tokens
- Returns user data

---

## 🌐 Step 7: Auth Server (30 min)

Create `services/auth/src/config/database.ts`:
```typescript
import mongoose from 'mongoose';

export const connectDB = async () => {
  try {
    const uri = process.env.MONGODB_URI || 'mongodb://admin:password@localhost:27017/myapp?authSource=admin';
    await mongoose.connect(uri);
    console.log('✅ MongoDB connected');
  } catch (error) {
    console.error('❌ MongoDB connection failed:', error);
    process.exit(1);
  }
};
```

Create `services/auth/src/server.ts`:
```typescript
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import dotenv from 'dotenv';
import { connectDB } from './config/database';
import authRoutes from './routes/auth';

dotenv.config();

const app = express();
const PORT = process.env.PORT || 4001;

app.use(helmet());
app.use(cors());
app.use(express.json());

app.use('/auth', authRoutes);

connectDB().then(() => {
  app.listen(PORT, () => {
    console.log(`🔐 Auth Service running on port ${PORT}`);
  });
});
```

### Test Auth Service
```bash
npm run dev
```

---

## 📦 Step 8: CRUD Service (2 hours)

### 8.1 Initialize
```bash
cd services/crud
npm init -y
npm install express mongoose zod cors helmet dotenv
npm install -D typescript @types/express @types/node ts-node nodemon
```

### 8.2 User Routes
Create `services/crud/src/routes/users.ts`:
```typescript
import express from 'express';
import { User } from '../models/User';

const router = express.Router();

// Get all users
router.get('/', async (req, res) => {
  try {
    const { page = 1, limit = 10 } = req.query;
    const users = await User.find()
      .limit(Number(limit))
      .skip((Number(page) - 1) * Number(limit))
      .populate('roles');
    
    const total = await User.countDocuments();
    
    res.json({
      users,
      pagination: {
        page: Number(page),
        limit: Number(limit),
        total,
        pages: Math.ceil(total / Number(limit))
      }
    });
  } catch (error) {
    res.status(500).json({ error: 'Server error' });
  }
});

// Create user
router.post('/', async (req, res) => {
  try {
    const user = await User.create(req.body);
    res.status(201).json({ message: 'User created', user });
  } catch (error) {
    res.status(500).json({ error: 'Server error' });
  }
});

export default router;
```

---

## 🚪 Step 9: API Gateway (1 hour)

### 9.1 Initialize
```bash
cd services/gateway
npm init -y
npm install express http-proxy-middleware cors helmet express-rate-limit
npm install -D typescript @types/express ts-node nodemon
```

### 9.2 Gateway Server
Create `services/gateway/src/server.ts`:
```typescript
import express from 'express';
import { createProxyMiddleware } from 'http-proxy-middleware';
import cors from 'cors';

const app = express();
const PORT = 3000;

app.use(cors());
app.use(express.json());

// Proxy to Auth Service
app.use('/api/auth', createProxyMiddleware({
  target: 'http://localhost:4001',
  pathRewrite: { '^/api/auth': '/auth' },
  changeOrigin: true
}));

// Proxy to CRUD Service
app.use('/api/users', createProxyMiddleware({
  target: 'http://localhost:4002',
  pathRewrite: { '^/api/users': '/users' },
  changeOrigin: true
}));

app.listen(PORT, () => {
  console.log(`🌐 API Gateway on port ${PORT}`);
});
```

---

## ⚛️ Step 10: React Frontend (4 hours)

### 10.1 Create React App
```bash
cd frontend
npm create vite@latest . -- --template react-ts
npm install
npm install axios react-router-dom react-query @tanstack/react-query
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### 10.2 API Client
Create `frontend/src/lib/api.ts`:
```typescript
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:3000/api'
});

// Add token to requests
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export const authAPI = {
  login: (email: string, password: string) =>
    api.post('/auth/login', { email, password })
};

export const usersAPI = {
  getAll: (params?: any) => api.get('/users', { params }),
  create: (data: any) => api.post('/users', data)
};
```

### 10.3 Login Page
Create `frontend/src/pages/LoginPage.tsx`:
```typescript
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { authAPI } from '../lib/api';

export default function LoginPage() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const navigate = useNavigate();

  const handleLogin = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      const { data } = await authAPI.login(email, password);
      localStorage.setItem('accessToken', data.accessToken);
      navigate('/dashboard');
    } catch (error) {
      alert('Login failed');
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center">
      <form onSubmit={handleLogin} className="bg-white p-8 rounded shadow">
        <h1 className="text-2xl mb-4">Login</h1>
        <input
          type="email"
          placeholder="Email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          className="w-full p-2 border mb-4"
        />
        <input
          type="password"
          placeholder="Password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          className="w-full p-2 border mb-4"
        />
        <button type="submit" className="w-full bg-blue-500 text-white p-2">
          Login
        </button>
      </form>
    </div>
  );
}
```

---

## 🐳 Step 11: Dockerize Everything (2 hours)

### 11.1 Auth Service Dockerfile
Create `services/auth/Dockerfile`:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
CMD ["npm", "start"]
```

### 11.2 Docker Compose
Create `docker-compose.microservices.yml`:
```yaml
version: '3.8'
services:
  mongodb:
    image: mongo:7
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password

  auth:
    build: ./services/auth
    ports:
      - "4001:4001"
    environment:
      MONGODB_URI: mongodb://admin:password@mongodb:27017/myapp?authSource=admin
    depends_on:
      - mongodb

  crud:
    build: ./services/crud
    ports:
      - "4002:4002"
    environment:
      MONGODB_URI: mongodb://admin:password@mongodb:27017/myapp?authSource=admin
    depends_on:
      - mongodb

  gateway:
    build: ./services/gateway
    ports:
      - "3000:3000"
    depends_on:
      - auth
      - crud

  frontend:
    build: ./frontend
    ports:
      - "5173:5173"
```

### 11.3 Start Everything
```bash
docker-compose -f docker-compose.microservices.yml up -d
```

---

## ✅ Step 12: Testing (1 hour)

### Test Login
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@test.com","password":"password123"}'
```

### Test Users
```bash
curl http://localhost:3000/api/users \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

## 🎯 What You've Built

✅ **5 Microservices**
- MongoDB database
- Auth service (JWT)
- CRUD service
- API Gateway
- React frontend

✅ **Key Features**
- User authentication
- Token-based security
- User management
- Microservices architecture
- Docker deployment

---

## 📚 Next Steps

1. **Add Role-Based Access Control**
    - Create Role and Permission models
    - Add middleware to check permissions
    - Protect routes based on roles

2. **Add More Features**
    - Activity logging
    - Dashboard with analytics
    - Search and filters
    - Email notifications

3. **Improve Security**
    - Add rate limiting
    - Implement refresh token rotation
    - Add input validation
    - Set up HTTPS

4. **Deploy to Production**
    - Use environment variables
    - Set up CI/CD
    - Deploy to cloud (AWS, Azure, etc.)
    - Add monitoring and logging

---

## 🆘 Common Issues

**MongoDB connection failed:**
```bash
# Check MongoDB is running
docker ps | grep mongodb

# Restart MongoDB
docker restart mongodb
```

**Port already in use:**
```bash
# Find process
netstat -ano | findstr :3000

# Kill process
taskkill /PID <pid> /F
```

**Build errors:**
```bash
# Clean install
rm -rf node_modules package-lock.json
npm install
```

---

## 📖 Learning Resources

- **React**: https://react.dev
- **TypeScript**: https://typescriptlang.org
- **Express**: https://expressjs.com
- **MongoDB**: https://mongodb.com/docs
- **Docker**: https://docs.docker.com
- **JWT**: https://jwt.io

---

**Congratulations! You've built a complete microservices application! 🎉**

*This tutorial teaches you the fundamentals. Keep building and improving!*
