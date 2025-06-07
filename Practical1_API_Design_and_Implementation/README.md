# RESTful API for Social Media Platform

## Overview
A comprehensive RESTful API implementation for a social media platform similar to Instagram, built with Node.js and Express. This project demonstrates proper API design principles, HTTP methods, status codes, content negotiation, and comprehensive documentation.

## Technologies Used
- Node.js
- Express.js
- Morgan (HTTP request logger)
- CORS (Cross-Origin Resource Sharing)
- Helmet (Security middleware)
- Nodemon (Development tool)
- JavaScript ES6+

## Project Setup

### Prerequisites
- Node.js (v14 or higher)
- npm or yarn package manager
- Git for version control
- API testing tool (Postman, Thunder Client, or curl)

### Installation Steps

1. **Create project directory**
   ```bash
   mkdir social-media-api
   cd social-media-api
   ```

2. **Initialize Node.js project**
   ```bash
   npm init -y
   ```

3. **Install dependencies**
   ```bash
   # Production dependencies
   npm install express morgan cors helmet
   
   # Development dependencies
   npm install nodemon --save-dev
   ```

4. **Create project structure**
   ```bash
   mkdir -p controllers routes middleware config utils public
   touch server.js .env .gitignore
   ```

5. **Configure environment variables**
   ```bash
   # .env file
   PORT=3000
   NODE_ENV=development
   ```

6. **Set up package.json scripts**
   ```json
   {
     "scripts": {
       "start": "node server.js",
       "dev": "nodemon server.js"
     }
   }
   ```

## API Resources and Endpoints

### Core Resources
- **Users**: User profiles and authentication
- **Posts**: Content posts with images and captions
- **Comments**: Comments on posts
- **Likes**: Like functionality for posts
- **Followers**: User following relationships

### Complete API Endpoints

#### Users Resource
| HTTP Method | Endpoint | Description |
|-------------|----------|-------------|
| GET | `/api/users` | Get all users (paginated) |
| GET | `/api/users/:id` | Get specific user by ID |
| POST | `/api/users` | Create new user |
| PUT | `/api/users/:id` | Update user profile |
| DELETE | `/api/users/:id` | Delete user account |

#### Posts Resource
| HTTP Method | Endpoint | Description |
|-------------|----------|-------------|
| GET | `/api/posts` | Get all posts (paginated) |
| GET | `/api/posts/:id` | Get specific post by ID |
| POST | `/api/posts` | Create new post |
| PUT | `/api/posts/:id` | Update post |
| DELETE | `/api/posts/:id` | Delete post |

#### Comments Resource
| HTTP Method | Endpoint | Description |
|-------------|----------|-------------|
| GET | `/api/posts/:postId/comments` | Get comments for a post |
| GET | `/api/comments/:id` | Get specific comment |
| POST | `/api/posts/:postId/comments` | Create comment on post |
| PUT | `/api/comments/:id` | Update comment |
| DELETE | `/api/comments/:id` | Delete comment |

#### Likes Resource
| HTTP Method | Endpoint | Description |
|-------------|----------|-------------|
| GET | `/api/posts/:postId/likes` | Get likes for a post |
| POST | `/api/posts/:postId/likes` | Like a post |
| DELETE | `/api/posts/:postId/likes` | Unlike a post |

#### Followers Resource
| HTTP Method | Endpoint | Description |
|-------------|----------|-------------|
| GET | `/api/users/:userId/followers` | Get user's followers |
| GET | `/api/users/:userId/following` | Get users that user follows |
| POST | `/api/users/:userId/follow` | Follow a user |
| DELETE | `/api/users/:userId/follow` | Unfollow a user |

## Project Structure

```
social-media-api/
├── controllers/
│   ├── userController.js       # User CRUD operations
│   ├── postController.js       # Post CRUD operations
│   ├── commentController.js    # Comment operations
│   ├── likeController.js       # Like operations
│   └── followerController.js   # Follow/unfollow operations
├── routes/
│   ├── users.js               # User route definitions
│   ├── posts.js               # Post route definitions
│   ├── comments.js            # Comment route definitions
│   ├── likes.js               # Like route definitions
│   └── followers.js           # Follower route definitions
├── middleware/
│   ├── errorHandler.js        # Global error handling
│   ├── async.js               # Async wrapper utility
│   └── formatResponse.js      # Content negotiation
├── utils/
│   ├── mockData.js           # Sample data for testing
│   └── errorResponse.js      # Custom error class
├── public/
│   └── docs.html             # API documentation
├── config/
│   └── database.js           # Database configuration (future)
├── server.js                 # Main application entry point
├── .env                      # Environment variables
├── .gitignore               # Git ignore rules
└── package.json             # Project dependencies
```

## Request/Response Examples

### Create User (POST /api/users)

**Request:**
```json
{
  "username": "new_traveler",
  "email": "new@example.com",
  "password": "securepassword",
  "full_name": "New Traveler",
  "bio": "Adventure seeker"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": 51,
    "username": "new_traveler",
    "full_name": "New Traveler",
    "email": "new@example.com",
    "bio": "Adventure seeker",
    "created_at": "2023-03-20T10:30:00Z"
  }
}
```

### Get Users with Pagination (GET /api/users)

**Request:**
```
GET /api/users?page=1&limit=10
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "success": true,
  "count": 50,
  "page": 1,
  "total_pages": 5,
  "data": [
    {
      "id": 1,
      "username": "traveler",
      "full_name": "Karma",
      "profile_picture": "https://example.com/profiles/alex.jpg",
      "bio": "Travel photographer",
      "created_at": "2023-01-15T08:00:00Z"
    }
  ]
}
```

### Create Post (POST /api/posts)

**Request:**
```json
{
  "user_id": 1,
  "image_url": "https://example.com/images/sunset.jpg",
  "caption": "Beautiful sunset at the beach",
  "tags": ["#sunset", "#beach", "#photography"]
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": 101,
    "user_id": 1,
    "image_url": "https://example.com/images/sunset.jpg",
    "caption": "Beautiful sunset at the beach",
    "tags": ["#sunset", "#beach", "#photography"],
    "likes_count": 0,
    "comments_count": 0,
    "created_at": "2023-03-20T15:45:00Z"
  }
}
```

## HTTP Status Codes Used

### Success Codes
- **200 OK**: Successful GET, PUT requests
- **201 Created**: Successful POST requests
- **204 No Content**: Successful DELETE requests

### Client Error Codes
- **400 Bad Request**: Invalid request data
- **401 Unauthorized**: Missing or invalid authentication
- **403 Forbidden**: Insufficient permissions
- **404 Not Found**: Resource not found
- **409 Conflict**: Resource already exists

### Server Error Codes
- **500 Internal Server Error**: Unexpected server errors

## Content Negotiation

The API supports multiple content types:

### Supported MIME Types
- `application/json` (default)
- `application/xml`
- `text/plain`
- `text/html`

### Example Usage
```bash
# JSON response (default)
curl -H "Accept: application/json" http://localhost:3000/api/users

# XML response
curl -H "Accept: application/xml" http://localhost:3000/api/users

# Plain text response
curl -H "Accept: text/plain" http://localhost:3000/api/users
```

## Error Handling

### Error Response Format
```json
{
  "success": false,
  "error": {
    "message": "User not found",
    "status": 404,
    "timestamp": "2023-03-20T10:30:00Z"
  }
}
```

### Custom Error Classes
- **ErrorResponse**: Custom error class with status codes
- **Global Error Handler**: Centralized error processing
- **Async Handler**: Wrapper for async route handlers

## Security Features

### Implemented Security Measures
- **Helmet**: Security headers
- **CORS**: Cross-origin resource sharing
- **Input Validation**: Request data validation
- **Error Information**: Sanitized error responses

### Headers Added by Helmet
- Content Security Policy
- X-Content-Type-Options
- X-Frame-Options
- X-XSS-Protection

## Mock Data Structure

### Users Mock Data
```javascript
const users = [
  {
    id: 1,
    username: "traveler",
    email: "traveler@example.com",
    full_name: "Karma",
    profile_picture: "https://example.com/profiles/alex.jpg",
    bio: "Travel photographer",
    created_at: "2023-01-15T08:00:00Z"
  }
];
```

### Posts Mock Data
```javascript
const posts = [
  {
    id: 1,
    user_id: 1,
    image_url: "https://example.com/images/post1.jpg",
    caption: "Amazing sunset view!",
    tags: ["#sunset", "#nature"],
    likes_count: 25,
    comments_count: 5,
    created_at: "2023-02-01T12:00:00Z"
  }
];
```

## API Documentation

### Interactive Documentation
- Access documentation at: `http://localhost:3000/docs`
- HTML-based documentation with examples
- Endpoint descriptions and usage instructions

### Documentation Features
- Complete endpoint list
- Request/response examples
- HTTP status code explanations
- Authentication requirements
- Parameter descriptions

## Testing the API

### Using curl
```bash
# Test user creation
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"username":"testuser","email":"test@example.com","password":"password123"}'

# Test getting users
curl -X GET http://localhost:3000/api/users

# Test getting specific user
curl -X GET http://localhost:3000/api/users/1
```

### Using Postman
1. Import the API endpoints
2. Set base URL to `http://localhost:3000`
3. Test each endpoint with sample data
4. Verify response formats and status codes

## Development Workflow

### Starting the Server
```bash
# Development mode with auto-reload
npm run dev

# Production mode
npm start
```

### Adding New Endpoints
1. Create controller function in appropriate controller file
2. Add route definition in corresponding route file
3. Test endpoint with sample data
4. Update documentation

## Performance Considerations

### Implemented Optimizations
- **Pagination**: Limit data returned per request
- **Selective Fields**: Return only necessary data
- **Caching Headers**: HTTP caching for static responses
- **Compression**: Gzip compression for responses

### Future Optimizations
- Database indexing for faster queries
- Redis caching for frequently accessed data
- Rate limiting to prevent abuse
- Database connection pooling

## Validation and Data Integrity

### Input Validation
- Required field validation
- Data type validation
- Format validation (email, URLs)
- Length constraints

### Business Logic Validation
- Unique username/email constraints
- User authorization for operations
- Relationship integrity (user exists before creating post)

## Future Enhancements

### Planned Features
- JWT authentication implementation
- File upload for images
- Real-time notifications
- Search functionality
- Advanced filtering and sorting
- API versioning
- Rate limiting
- Comprehensive logging

### Database Integration
- MongoDB/PostgreSQL integration
- Data persistence
- Transaction support
- Migration scripts

This RESTful API implementation demonstrates industry best practices for API design, proper HTTP method usage, comprehensive error handling, and thorough documentation.