# Practical 4: Connecting TikTok to PostgreSQL with Prisma ORM

## Table of Contents
- [Objectives](#objectives)
- [Prerequisites](#prerequisites)
- [Part 1: Setting Up PostgreSQL Database](#part-1-setting-up-postgresql-database)
- [Part 2: Setting Up Prisma ORM](#part-2-setting-up-prisma-orm)
- [Part 3: Creating the Database Schema](#part-3-creating-the-database-schema)
- [Part 4: Update Controllers to Use Prisma](#part-4-update-controllers-to-use-prisma)
- [Part 5: Implementing Authentication](#part-5-implementing-authentication)
- [Part 6: Testing Database Integration](#part-6-testing-database-integration)
- [Part 7: Creating Test Data](#part-7-creating-test-data)
- [Key Concepts](#key-concepts)
- [Resources](#resources)

## Objectives
- Set up a PostgreSQL database for our TikTok clone application
- Configure Prisma ORM to interact with the database
- Migrate from in-memory data models to persistent database storage
- Implement authentication with password encryption
- Update our RESTful API endpoints to use the database

## Prerequisites
- Node.js installed
- PostgreSQL installed and running
- Basic understanding of SQL and databases
- Completed previous TikTok clone practicals

## Part 1: Setting Up PostgreSQL Database

### Step 1: Create a Database

1. **Access PostgreSQL command line:**
   ```bash
   sudo -u postgres psql
   ```

2. **Create a new database:**
   ```sql
   CREATE DATABASE tiktok_db;
   ```

3. **Create a user for the application:**
   ```sql
   CREATE USER tiktok_user WITH ENCRYPTED PASSWORD 'your_password';
   GRANT ALL PRIVILEGES ON DATABASE tiktok_db TO tiktok_user;
   ```

4. **Exit PostgreSQL command line:**
   ```sql
   \q
   ```

## Part 2: Setting Up Prisma ORM

### Step 1: Install Prisma Dependencies

Navigate to your server project and install Prisma:
```bash
cd server
npm install @prisma/client
npm install prisma --save-dev
```

### Step 2: Initialize Prisma

```bash
npx prisma init
```

This creates:
- A `prisma` directory with a `schema.prisma` file
- A `.env` file for environment variables

### Step 3: Configure Database Connection

Edit the `.env` file:
```env
DATABASE_URL="postgresql://tiktok_user:your_password@localhost:5432/tiktok_db?schema=public"
```

### Step 4: Define Prisma Schema

Replace the contents of `prisma/schema.prisma` with the TikTok data model schema.

### Step 5: Install Additional Dependencies

```bash
npm install bcrypt jsonwebtoken
```

## Part 3: Creating the Database Schema

### Step 1: Create Prisma Migration

```bash
npx prisma migrate dev --name init
```

This command:
1. Creates SQL migration files in `prisma/migrations` directory
2. Applies the migration to your database
3. Generates the Prisma Client

### Step 2: Create Prisma Client Instance

Create `src/lib/prisma.js`:
```javascript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

export default prisma
```

### Step 3: Create Authentication Middleware

Create `src/middleware/auth.js` with JWT token verification logic.


## Part 4: Update Controllers to Use Prisma

**Reference Repository:** [TikTok Server GitHub](https://github.com/syangche/TikTok_Server.git)

### Step 1: Update User Controller

Update `src/controllers/userController.js` to use Prisma:

**Key concepts implemented:**
- Password hashing using bcrypt
- JWT token generation for authentication
- Database queries using Prisma Client

### Step 2: Update Video Controller

Update `src/controllers/videoController.js`:

**Key concepts implemented:**
- Complex queries with relationships
- Transactions for operations affecting multiple tables
- Using Prisma's count and aggregation features

### Step 3: Update Comment Controller

Update `src/controllers/commentController.js` with Prisma integration.

### Step 4: Update app.js

Register all routes in the main application file.

## Part 5: Implementing Authentication

### Step 1: Create Authentication Middleware

Create `src/middleware/auth.js` with JWT token verification functionality.

### Step 2: Update Routes

Update route files to use authentication middleware for protected routes.

### Step 3: Configure Environment Variables

```env
# Server settings
PORT=5000
NODE_ENV=development

# Database settings
DATABASE_URL="postgresql://tiktok_user:your_password@localhost:5432/tiktok_db?schema=public"

# JWT settings
JWT_SECRET=yourverylongandsecurerandomsecret
JWT_EXPIRE=30d
```

## Part 6: Testing Database Integration

### Step 1: Start the Server

```bash
npm run dev
```

### Step 2: Test Registration and Login

Use Postman to test:
- Register a new user
- Login with credentials

### Step 3: Test Protected Routes

Use the JWT token from login to test protected routes:
- Create a video (protected route)
- Update user profile
- Delete content

**Postman Guide:** [Testing Guide](https://docs.google.com/document/d/1OlnYRUqXZYWUl5AksoGOQYFqaT7
1KYJ6wiDU03y40Fk/edit?usp=sharing)

## Part 7: Creating Test Data

### Step 1: Create Seed File

Create `prisma/seed.js` with test data generation logic.

### Step 2: Add Script to package.json

```json
{
  "scripts": {
    "dev": "nodemon src/index.js",
    "start": "node src/index.js",
    "seed": "node prisma/seed.js"
  }
}
```

### Step 3: Install Dependencies

```bash
npm install bcrypt
```

### Step 4: Run Seed Script

```bash
npm run seed
```

This populates the database with:
- 10 users
- 50 videos (5 per user)
- 200 comments
- 300 video likes
- 150 comment likes
- 40 follow relationships

## Key Concepts

### Database Schema Design
- **Tables:** Represent entities like users, videos, comments
- **Relationships:** Connections between tables (one-to-many, many-to-many)
- **Indexes:** Improve query performance
- **Foreign Keys:** Maintain data integrity

### Object-Relational Mapping (ORM)
- Maps database tables to programming objects
- Simplifies database operations with type safety
- Reduces boilerplate SQL code
- Handles database migrations

### Authentication and Security
- **Password Hashing:** Never store plain-text passwords
- **JWT Tokens:** Secure, stateless authentication
- **Protected Routes:** Middleware to secure endpoints

### Prisma Specific Features
- **Model Definitions:** Define data structure in schema.prisma
- **Migrations:** Version control for database schema
- **Relationships:** Define connections between models
- **Transactions:** Ensure data consistency across operations

## Resources

- [Prisma Documentation](https://www.prisma.io/docs)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [JWT Authentication Guide](https://jwt.io/introduction)
- [Bcrypt Documentation](https://www.npmjs.com/package/bcrypt)
