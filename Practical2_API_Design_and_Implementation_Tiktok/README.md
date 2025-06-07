# TikTok REST API Design and Implementation

## Overview
This project implements a RESTful API for a TikTok-like application using Node.js and Express.js. The API provides endpoints for managing videos, users, and comments with full CRUD operations.

## Project Structure
```
server/
├── src/
│   ├── controllers/
│   │   ├── videoController.js
│   │   ├── userController.js
│   │   └── commentController.js
│   ├── routes/
│   │   ├── videos.js
│   │   ├── users.js
│   │   └── comments.js
│   ├── models/
│   │   └── index.js
│   ├── middleware/
│   ├── utils/
│   ├── app.js
│   └── server.js
├── .env
└── package.json
```

## API Design

### Resources
- **Videos**: Core content of the platform
- **Users**: Platform users and their profiles
- **Comments**: User interactions on videos

### Endpoint Overview

| Endpoint | GET | POST | PUT | DELETE |
|----------|-----|------|-----|---------|
| `/api/videos` | Get all videos | Create new video | - | - |
| `/api/videos/:id` | Get video by ID | - | Update video | Delete video |
| `/api/videos/:id/comments` | Get video comments | - | - | - |
| `/api/videos/:id/likes` | Get video likes | Like video | - | Unlike video |
| `/api/users` | Get all users | Create user | - | - |
| `/api/users/:id` | Get user by ID | - | Update user | Delete user |
| `/api/users/:id/videos` | Get user videos | - | - | - |
| `/api/users/:id/followers` | Get followers | Follow user | - | Unfollow user |
| `/api/users/:id/following` | Get following users | - | - | - |
| `/api/comments` | Get all comments | Create comment | - | - |
| `/api/comments/:id` | Get comment by ID | - | Update comment | Delete comment |
| `/api/comments/:id/likes` | Get comment likes | Like comment | - | Unlike comment |

## Installation & Setup

### Step 1: Initialize Project
```bash
mkdir -p server
cd server
npm init -y
```

### Step 2: Install Dependencies
```bash
# Production dependencies
npm install express cors morgan body-parser dotenv

# Development dependencies
npm install --save-dev nodemon
```

#### Dependencies Overview:
- **express**: Web server framework for Node.js
- **cors**: Enable Cross-Origin Resource Sharing
- **morgan**: HTTP request logger middleware
- **body-parser**: Parse incoming request bodies
- **dotenv**: Load environment variables from .env file
- **nodemon**: Auto-restart server during development

### Step 3: Create Project Structure
```bash
mkdir -p src/controllers src/routes src/models src/middleware src/utils
touch src/app.js src/server.js .env
```

### Step 4: Environment Configuration
Create `.env` file:
```env
PORT=3000
NODE_ENV=development
```

### Step 5: Application Setup

#### src/app.js
```javascript
const express = require('express');
const cors = require('cors');
const morgan = require('morgan');
const bodyParser = require('body-parser');
require('dotenv').config();

// Import routes
const videoRoutes = require('./routes/videos');
const userRoutes = require('./routes/users');
const commentRoutes = require('./routes/comments');

const app = express();

// Middleware
app.use(cors());
app.use(morgan('combined'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

// Routes
app.use('/api/videos', videoRoutes);
app.use('/api/users', userRoutes);
app.use('/api/comments', commentRoutes);

// Health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({ message: 'Server is running successfully!' });
});

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({ error: 'Route not found' });
});

module.exports = app;
```

#### src/server.js
```javascript
const app = require('./app');

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
  console.log(`Environment: ${process.env.NODE_ENV}`);
});
```

### Step 6: Package.json Scripts
Update the scripts section in `package.json`:
```json
{
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  }
}
```

## Implementation Details

### Data Models (In-Memory Storage)
Located in `src/models/index.js`, containing sample data for:
- Videos with metadata (title, description, user, likes, comments)
- Users with profile information and relationships
- Comments with user references and content

### Controllers Implementation

#### Video Controller (`src/controllers/videoController.js`)
- `getAllVideos()`: Retrieve all videos
- `getVideoById()`: Get specific video by ID
- `createVideo()`: Add new video
- `updateVideo()`: Modify existing video
- `deleteVideo()`: Remove video
- `getVideoComments()`: Get comments for a video
- `likeVideo()`: Add like to video
- `unlikeVideo()`: Remove like from video

#### User Controller (`src/controllers/userController.js`)
- `getAllUsers()`: Retrieve all users
- `getUserById()`: Get specific user by ID
- `createUser()`: Register new user
- `updateUser()`: Modify user profile
- `deleteUser()`: Remove user account
- `getUserVideos()`: Get videos by user
- `followUser()`: Follow another user
- `unfollowUser()`: Unfollow user
- `getFollowers()`: Get user's followers
- `getFollowing()`: Get users being followed

#### Comment Controller (`src/controllers/commentController.js`)
- `getAllComments()`: Retrieve all comments
- `getCommentById()`: Get specific comment
- `createComment()`: Add new comment
- `updateComment()`: Modify comment
- `deleteComment()`: Remove comment
- `likeComment()`: Like a comment
- `unlikeComment()`: Remove like from comment

### Routes Implementation
- `src/routes/videos.js`: Video-related endpoints
- `src/routes/users.js`: User-related endpoints  
- `src/routes/comments.js`: Comment-related endpoints

## Testing the API

### Running the Server
```bash
# Development mode
npm run dev

# Production mode
npm start
```

### API Testing Commands

#### Users Endpoints
```bash
# Get all users
curl -X GET http://localhost:3000/api/users

# Get user by ID
curl -X GET http://localhost:3000/api/users/1

# Create new user
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"username":"newuser","email":"user@example.com"}'
```

#### Videos Endpoints
```bash
# Get all videos
curl -X GET http://localhost:3000/api/videos

# Get video by ID
curl -X GET http://localhost:3000/api/videos/1

# Get user's videos
curl -X GET http://localhost:3000/api/users/1/videos

# Get video comments
curl -X GET http://localhost:3000/api/videos/1/comments
```

#### Comments Endpoints
```bash
# Get all comments
curl -X GET http://localhost:3000/api/comments

# Get comment by ID
curl -X GET http://localhost:3000/api/comments/1
```

### Expected Response Format
All API responses follow this structure:
```json
{
  "success": true,
  "data": {},
  "message": "Operation successful"
}
```

Error responses:
```json
{
  "success": false,
  "error": "Error message",
  "statusCode": 400
}
```

## Features Implemented
- ✅ RESTful API design principles
- ✅ CRUD operations for all resources
- ✅ Proper HTTP status codes
- ✅ Error handling middleware
- ✅ CORS configuration
- ✅ Request logging
- ✅ Environment configuration
- ✅ Modular code structure
- ✅ In-memory data storage

## Next Steps
- [ ] Database integration (MongoDB/PostgreSQL)
- [ ] Authentication & Authorization
- [ ] Input validation
- [ ] Rate limiting
- [ ] API documentation (Swagger)
- [ ] Unit testing
- [ ] File upload for videos
- [ ] Real-time features (WebSocket)

## Technologies Used
- **Node.js**: Runtime environment
- **Express.js**: Web framework
- **JavaScript ES6+**: Programming language
- **npm**: Package manager

## Author
[Your Name]

## License
This project is for educational purposes.