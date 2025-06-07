# Authentication System - JWT Token Implementation 🔐

A comprehensive guide to implementing secure authentication using TypeScript, Express.js, MongoDB, and JWT tokens with email/password validation.

## Overview

This project demonstrates a complete authentication system featuring:
- **User Registration & Login**: Secure email/password authentication
- **JWT Token Management**: Stateless authentication with access tokens
- **Password Security**: Bcrypt hashing with salt rounds
- **Database Integration**: MongoDB with Mongoose ODM
- **Input Validation**: Comprehensive request validation
- **Error Handling**: Secure error responses

### Key Features
- ✅ User registration with email uniqueness validation
- ✅ Secure password hashing using bcrypt (12 salt rounds)
- ✅ JWT access token generation and verification
- ✅ Protected route middleware
- ✅ Comprehensive error handling
- ✅ User profile management
- ✅ Account status tracking
- ✅ Request validation middleware

## Tech Stack

### Backend Technologies
- **Runtime**: Node.js v18+
- **Framework**: Express.js v4.18+
- **Database**: MongoDB v6.0+
- **ODM**: Mongoose v7.0+
- **Authentication**: JWT (jsonwebtoken)
- **Password Hashing**: bcryptjs
- **Validation**: express-validator
- **Environment**: dotenv

### Development Tools
- **Language**: TypeScript v5.0+
- **Package Manager**: npm/yarn
- **Code Editor**: Visual Studio Code (recommended)
- **API Testing**: Postman/Thunder Client

## Quick Start

### 1. Clone & Install

```bash
git clone <repository-url>
cd authentication-jwt-system
npm install
```

### 2. Environment Setup

Create `.env` file in root directory:

```env
# Server Configuration
PORT=3000
NODE_ENV=development

# Database
MONGODB_URI=mongodb://localhost:27017/auth_system
DB_NAME=authentication_db

# JWT Configuration
JWT_SECRET=your-super-secret-jwt-key-change-in-production
JWT_EXPIRES_IN=24h

# Security
BCRYPT_SALT_ROUNDS=12
```

### 3. Database Setup

Start MongoDB service:

```bash
# Windows
net start MongoDB

# macOS (via Homebrew)
brew services start mongodb-community

# Linux (systemd)
sudo systemctl start mongod
```

### 4. Development Server

```bash
# Development mode with hot reload
npm run dev

# Production build
npm run build
npm start
```

Server runs on `http://localhost:3000`

## Database Schema

### User Model
```typescript
interface IUser {
  _id: ObjectId;
  email: string;           // Unique email address
  password: string;        // Bcrypt hashed password
  firstName: string;       // User's first name
  lastName: string;        // User's last name
  isActive: boolean;       // Account status
  lastLogin?: Date;        // Last login timestamp
  createdAt: Date;         // Account creation date
  updatedAt: Date;         // Last modification date
}
```

### Validation Rules
- **Email**: Valid email format, unique across system
- **Password**: Minimum 8 characters, must contain uppercase, lowercase, number
- **Names**: 2-50 characters, alphabetic only
- **IsActive**: Boolean flag for account status

## 🛡 API Endpoints

### Authentication Routes

| Method | Endpoint | Description | Request Body | Response |
|--------|----------|-------------|--------------|----------|
| POST | `/api/auth/register` | User registration | `{ email, password, firstName, lastName }` | User data + JWT |
| POST | `/api/auth/login` | User login | `{ email, password }` | User data + JWT |
| POST | `/api/auth/logout` | User logout | - | Success message |

### Protected Routes (Require JWT)

| Method | Endpoint | Description | Headers | Response |
|--------|----------|-------------|---------|----------|
| GET | `/api/users/profile` | Get user profile | `Authorization: Bearer <token>` | User profile |
| PUT | `/api/users/profile` | Update profile | `Authorization: Bearer <token>` | Updated profile |
| GET | `/api/users/dashboard` | User dashboard | `Authorization: Bearer <token>` | Dashboard data |

## Authentication Flow

### Registration Process
```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant Database

    Client->>Server: POST /api/auth/register
    Server->>Server: Validate input data
    Server->>Database: Check email uniqueness
    Server->>Server: Hash password (bcrypt)
    Server->>Database: Create user record
    Server->>Server: Generate JWT token
    Server->>Client: Return user data + JWT
```

### Login Process
```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant Database

    Client->>Server: POST /api/auth/login
    Server->>Database: Find user by email
    Server->>Server: Verify password (bcrypt.compare)
    Server->>Database: Update lastLogin timestamp
    Server->>Server: Generate JWT token
    Server->>Client: Return user data + JWT
```

### Protected Route Access
```mermaid
sequenceDiagram
    participant Client
    participant Middleware
    participant Server
    participant Database

    Client->>Middleware: Request with JWT token
    Middleware->>Middleware: Verify JWT signature
    Middleware->>Database: Validate user exists
    Middleware->>Server: Forward authenticated request
    Server->>Client: Return protected data
```

## Implementation Details

### 1. User Registration

```typescript
export const register = async (req: Request, res: Response): Promise<void> => {
  try {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors: errors.array()
      });
      return;
    }

    const { email, password, firstName, lastName } = req.body;

    // Check if user already exists
    const existingUser = await User.findOne({ email: email.toLowerCase() });
    if (existingUser) {
      res.status(409).json({
        success: false,
        message: 'User already exists with this email'
      });
      return;
    }

    // Hash password
    const saltRounds = parseInt(process.env.BCRYPT_SALT_ROUNDS || '12');
    const hashedPassword = await bcrypt.hash(password, saltRounds);

    // Create user
    const user = new User({
      email: email.toLowerCase(),
      password: hashedPassword,
      firstName,
      lastName,
      isActive: true
    });

    await user.save();

    // Generate JWT token
    const token = jwt.sign(
      { userId: user._id, email: user.email },
      process.env.JWT_SECRET!,
      { expiresIn: process.env.JWT_EXPIRES_IN || '24h' }
    );

    // Return response (exclude password)
    const userResponse = {
      id: user._id,
      email: user.email,
      firstName: user.firstName,
      lastName: user.lastName,
      isActive: user.isActive,
      createdAt: user.createdAt
    };

    res.status(201).json({
      success: true,
      message: 'User registered successfully',
      data: {
        user: userResponse,
        token
      }
    });

  } catch (error) {
    console.error('Registration error:', error);
    res.status(500).json({
      success: false,
      message: 'Internal server error'
    });
  }
};
```

### 2. User Login

```typescript
export const login = async (req: Request, res: Response): Promise<void> => {
  try {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors: errors.array()
      });
      return;
    }

    const { email, password } = req.body;

    // Find user by email
    const user = await User.findOne({ 
      email: email.toLowerCase(),
      isActive: true 
    });

    if (!user) {
      res.status(401).json({
        success: false,
        message: 'Invalid credentials'
      });
      return;
    }

    // Verify password
    const isPasswordValid = await bcrypt.compare(password, user.password);
    if (!isPasswordValid) {
      res.status(401).json({
        success: false,
        message: 'Invalid credentials'
      });
      return;
    }

    // Update last login
    user.lastLogin = new Date();
    await user.save();

    // Generate JWT token
    const token = jwt.sign(
      { userId: user._id, email: user.email },
      process.env.JWT_SECRET!,
      { expiresIn: process.env.JWT_EXPIRES_IN || '24h' }
    );

    // Return response (exclude password)
    const userResponse = {
      id: user._id,
      email: user.email,
      firstName: user.firstName,
      lastName: user.lastName,
      isActive: user.isActive,
      lastLogin: user.lastLogin
    };

    res.status(200).json({
      success: true,
      message: 'Login successful',
      data: {
        user: userResponse,
        token
      }
    });

  } catch (error) {
    console.error('Login error:', error);
    res.status(500).json({
      success: false,
      message: 'Internal server error'
    });
  }
};
```

### 3. JWT Authentication Middleware

```typescript
export const authenticateToken = async (
  req: AuthenticatedRequest,
  res: Response,
  next: NextFunction
): Promise<void> => {
  try {
    const authHeader = req.headers.authorization;
    const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN

    if (!token) {
      res.status(401).json({
        success: false,
        message: 'Access token required'
      });
      return;
    }

    // Verify JWT token
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as JwtPayload;
    
    // Check if user still exists and is active
    const user = await User.findById(decoded.userId).select('-password');
    if (!user || !user.isActive) {
      res.status(401).json({
        success: false,
        message: 'Invalid or expired token'
      });
      return;
    }

    // Attach user to request object
    req.user = user;
    next();

  } catch (error) {
    if (error instanceof jwt.JsonWebTokenError) {
      res.status(401).json({
        success: false,
        message: 'Invalid token'
      });
      return;
    }

    console.error('Authentication error:', error);
    res.status(500).json({
      success: false,
      message: 'Internal server error'
    });
  }
};
```

### 4. Input Validation

```typescript
export const registerValidation = [
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Please provide a valid email'),
  
  body('password')
    .isLength({ min: 8 })
    .withMessage('Password must be at least 8 characters long')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .withMessage('Password must contain at least one uppercase letter, one lowercase letter, and one number'),
  
  body('firstName')
    .trim()
    .isLength({ min: 2, max: 50 })
    .withMessage('First name must be between 2 and 50 characters')
    .matches(/^[A-Za-z\s]+$/)
    .withMessage('First name must contain only letters'),
  
  body('lastName')
    .trim()
    .isLength({ min: 2, max: 50 })
    .withMessage('Last name must be between 2 and 50 characters')
    .matches(/^[A-Za-z\s]+$/)
    .withMessage('Last name must contain only letters')
];

export const loginValidation = [
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Please provide a valid email'),
  
  body('password')
    .notEmpty()
    .withMessage('Password is required')
];
```

## Security Features

### Password Security
- **Bcrypt Hashing**: Industry-standard password hashing
- **Salt Rounds**: Configurable cost factor (default: 12)
- **Password Complexity**: Enforced minimum requirements
- **No Plain Text Storage**: Passwords never stored in readable format

### JWT Security
- **Secret Key Management**: Environment-based secret storage
- **Token Expiration**: Configurable expiration times
- **Payload Minimization**: Only essential data in token
- **Signature Verification**: Prevents token tampering

### Input Validation
- **Email Validation**: RFC-compliant email checking
- **Data Sanitization**: Input cleaning and normalization
- **Length Limits**: Prevent buffer overflow attacks
- **Type Validation**: Ensure correct data types

### Error Handling
- **Generic Error Messages**: Prevent information leakage
- **Consistent Responses**: Uniform error response format
- **Logging**: Comprehensive error logging for debugging
- **Rate Limiting**: (Recommended for production)

## 🧪 Testing

### Manual Testing with cURL

#### Register User
```bash
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "password": "SecurePass123",
    "firstName": "John",
    "lastName": "Doe"
  }'
```

#### Login User
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "password": "SecurePass123"
  }'
```

#### Access Protected Route
```bash
curl -X GET http://localhost:3000/api/users/profile \
  -H "Authorization: Bearer YOUR_JWT_TOKEN_HERE"
```

### Postman Collection

```json
{
  "info": {
    "name": "Authentication API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Register User",
      "request": {
        "method": "POST",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n  \"email\": \"test@example.com\",\n  \"password\": \"Password123\",\n  \"firstName\": \"Test\",\n  \"lastName\": \"User\"\n}"
        },
        "url": {
          "raw": "{{baseUrl}}/api/auth/register",
          "host": ["{{baseUrl}}"],
          "path": ["api", "auth", "register"]
        }
      }
    },
    {
      "name": "Login User",
      "request": {
        "method": "POST",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n  \"email\": \"test@example.com\",\n  \"password\": \"Password123\"\n}"
        },
        "url": {
          "raw": "{{baseUrl}}/api/auth/login",
          "host": ["{{baseUrl}}"],
          "path": ["api", "auth", "login"]
        }
      }
    },
    {
      "name": "Get Profile",
      "request": {
        "method": "GET",
        "header": [
          {
            "key": "Authorization",
            "value": "Bearer {{token}}"
          }
        ],
        "url": {
          "raw": "{{baseUrl}}/api/users/profile",
          "host": ["{{baseUrl}}"],
          "path": ["api", "users", "profile"]
        }
      }
    }
  ],
  "variable": [
    {
      "key": "baseUrl",
      "value": "http://localhost:3000"
    },
    {
      "key": "token",
      "value": "your-jwt-token-here"
    }
  ]
}
```

## Project Structure

```
authentication-jwt-system/
├── src/
│   ├── controllers/
│   │   ├── authController.ts      # Authentication logic
│   │   └── userController.ts      # User management
│   ├── middleware/
│   │   ├── auth.ts               # JWT authentication
│   │   └── validation.ts         # Input validation
│   ├── models/
│   │   └── User.ts              # User model schema
│   ├── routes/
│   │   ├── auth.ts              # Authentication routes
│   │   └── users.ts             # User routes
│   ├── types/
│   │   └── auth.ts              # TypeScript interfaces
│   ├── utils/
│   │   ├── database.ts          # Database connection
│   │   └── logger.ts            # Logging utility
│   ├── config/
│   │   └── database.ts          # Database configuration
│   └── app.ts                   # Express app setup
├── tests/
│   ├── auth.test.ts             # Authentication tests
│   └── users.test.ts            # User tests
├── .env.example                 # Environment template
├── .gitignore                   # Git ignore rules
├── package.json                 # Dependencies
├── tsconfig.json               # TypeScript config
└── README.md                   # This file
```

## Production Deployment

### Environment Variables (Production)
```env
NODE_ENV=production
PORT=443
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/production_db
JWT_SECRET=super-secure-production-secret-key-256-bits
JWT_EXPIRES_IN=1h
BCRYPT_SALT_ROUNDS=12
```

### Security Checklist
- [ ] Use HTTPS in production
- [ ] Implement rate limiting
- [ ] Add request logging
- [ ] Set up monitoring and alerting
- [ ] Configure CORS properly
- [ ] Use environment variables for secrets
- [ ] Implement refresh tokens
- [ ] Add API documentation
- [ ] Set up database backups
- [ ] Configure security headers

### Performance Optimization
- [ ] Database indexing on email field
- [ ] Connection pooling for MongoDB
- [ ] Implement caching (Redis)
- [ ] Compress responses
- [ ] Optimize JWT payload size
- [ ] Monitor response times
- [ ] Set up load balancing

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Acknowledgments

- [Express.js](https://expressjs.com/) - Web framework
- [MongoDB](https://www.mongodb.com/) - Database
- [JWT](https://jwt.io/) - Token authentication
- [bcrypt](https://github.com/kelektiv/node.bcrypt.js) - Password hashing
- [Mongoose](https://mongoosejs.com/) - MongoDB ODM