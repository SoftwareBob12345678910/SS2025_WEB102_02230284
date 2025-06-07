# File Upload Server Implementation

## Overview
This project implements a comprehensive server-side file upload system using Node.js and Express.js that can properly receive, validate, store, and serve files uploaded from a React/Next.js frontend. The implementation includes file type validation, size limitations, progress tracking, and proper error handling to create a secure and efficient file storage system.

## Objective
Implement a complete file upload flow from client to server in modern web applications by:
- Setting up Express server environment for file handling
- Configuring Multer middleware for multipart form data processing
- Creating secure upload endpoints with validation
- Implementing frontend-backend communication
- Testing the complete upload system

## Project Structure
```
file-upload-server/
├── uploads/           # Directory for uploaded files
├── server.js         # Main server file
├── .env              # Environment variables
├── package.json      # Project dependencies
└── README.md         # Project documentation
```

## Implementation Flow

### 1. Express Server Environment Setup
- Install necessary dependencies for file handling
- Create basic server structure for HTTP requests
- Configure middleware stack

### 2. Multer Middleware Configuration
- Process multipart form data
- Manage file storage on server
- Handle file metadata

### 3. Upload Endpoints Creation
- Accept file uploads via API routes
- Validate content types and file sizes
- Store files securely with proper naming

### 4. Validation and Security Implementation
- File type checking and filtering
- Size limit enforcement
- Comprehensive error handling
- Security vulnerability prevention

### 5. Frontend-Backend Connection
- Modify frontend to communicate with Express server
- Replace Next.js API routes with Express endpoints
- Handle cross-origin requests with CORS

### 6. Complete System Testing
- Verify file upload functionality
- Test validation and error scenarios
- Ensure proper file storage and access

## Installation & Setup

### Step 1: Initialize Project
```bash
mkdir file-upload-server
cd file-upload-server
npm init -y
```

### Step 2: Install Dependencies
```bash
npm install express cors multer morgan dotenv
```

#### Dependencies Overview:
- **express**: Web server framework for Node.js
- **cors**: Middleware to enable Cross-Origin Resource Sharing
- **multer**: Middleware for handling multipart/form-data (file uploads)
- **morgan**: HTTP request logger for debugging
- **dotenv**: Environment variable management

### Step 3: Environment Configuration
Create `.env` file:
```env
PORT=8000
NODE_ENV=development
FRONTEND_URL=http://localhost:3000
UPLOAD_PATH=./uploads
MAX_FILE_SIZE=10485760
```

## Server Implementation

### Basic Server Structure (`server.js`)
```javascript
const express = require('express');
const cors = require('cors');
const multer = require('multer');
const morgan = require('morgan');
const path = require('path');
const fs = require('fs');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 8000;

// Create uploads directory if it doesn't exist
const uploadDir = process.env.UPLOAD_PATH || './uploads';
if (!fs.existsSync(uploadDir)) {
    fs.mkdirSync(uploadDir, { recursive: true });
}

// Middleware
app.use(morgan('combined'));
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// CORS Configuration
app.use(cors({
    origin: process.env.FRONTEND_URL || 'http://localhost:3000',
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization']
}));

// Serve static files from uploads directory
app.use('/uploads', express.static(uploadDir));
```

### Multer Configuration
```javascript
// Multer storage configuration
const storage = multer.diskStorage({
    destination: function (req, file, cb) {
        cb(null, uploadDir);
    },
    filename: function (req, file, cb) {
        // Generate unique filename with timestamp
        const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
        cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
    }
});

// File filter for validation
const fileFilter = (req, file, cb) => {
    // Allowed file types
    const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf', 'text/plain'];
    
    if (allowedTypes.includes(file.mimetype)) {
        cb(null, true);
    } else {
        cb(new Error('Invalid file type. Only JPEG, PNG, GIF, PDF, and TXT files are allowed.'), false);
    }
};

// Multer configuration
const upload = multer({
    storage: storage,
    fileFilter: fileFilter,
    limits: {
        fileSize: parseInt(process.env.MAX_FILE_SIZE) || 10 * 1024 * 1024, // 10MB default
        files: 5 // Maximum 5 files per request
    }
});
```

### Upload API Endpoint
```javascript
// File upload endpoint
app.post('/api/upload', upload.array('files', 5), (req, res) => {
    try {
        if (!req.files || req.files.length === 0) {
            return res.status(400).json({
                success: false,
                message: 'No files uploaded'
            });
        }

        // Process uploaded files
        const uploadedFiles = req.files.map(file => ({
            originalName: file.originalname,
            filename: file.filename,
            size: file.size,
            mimetype: file.mimetype,
            path: `/uploads/${file.filename}`,
            uploadedAt: new Date().toISOString()
        }));

        res.status(200).json({
            success: true,
            message: `${req.files.length} file(s) uploaded successfully`,
            files: uploadedFiles
        });

    } catch (error) {
        console.error('Upload error:', error);
        res.status(500).json({
            success: false,
            message: 'Server error during file upload'
        });
    }
});

// Get uploaded files list
app.get('/api/files', (req, res) => {
    try {
        const files = fs.readdirSync(uploadDir).map(filename => {
            const filePath = path.join(uploadDir, filename);
            const stats = fs.statSync(filePath);
            
            return {
                filename,
                size: stats.size,
                uploadedAt: stats.birthtime.toISOString(),
                path: `/uploads/${filename}`
            };
        });

        res.json({
            success: true,
            files
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error retrieving files'
        });
    }
});
```

### Error Handling Middleware
```javascript
// Multer error handling
app.use((error, req, res, next) => {
    if (error instanceof multer.MulterError) {
        if (error.code === 'LIMIT_FILE_SIZE') {
            return res.status(400).json({
                success: false,
                message: 'File too large. Maximum size is 10MB.'
            });
        }
        if (error.code === 'LIMIT_FILE_COUNT') {
            return res.status(400).json({
                success: false,
                message: 'Too many files. Maximum 5 files allowed.'
            });
        }
        if (error.code === 'LIMIT_UNEXPECTED_FILE') {
            return res.status(400).json({
                success: false,
                message: 'Unexpected field name for file upload.'
            });
        }
    }
    
    if (error.message.includes('Invalid file type')) {
        return res.status(400).json({
            success: false,
            message: error.message
        });
    }

    console.error('Server error:', error);
    res.status(500).json({
        success: false,
        message: 'Internal server error'
    });
});

// 404 handler
app.use((req, res) => {
    res.status(404).json({
        success: false,
        message: 'Route not found'
    });
});

// Start server
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
    console.log(`Upload directory: ${uploadDir}`);
    console.log(`Frontend URL: ${process.env.FRONTEND_URL || 'http://localhost:3000'}`);
});
```

## Frontend Integration

### Updated React Component (page.js)
```javascript
'use client';
import { useState } from 'react';
import axios from 'axios';

export default function FileUpload() {
    const [files, setFiles] = useState([]);
    const [uploading, setUploading] = useState(false);
    const [uploadProgress, setUploadProgress] = useState(0);
    const [uploadedFiles, setUploadedFiles] = useState([]);

    const handleFileChange = (e) => {
        const selectedFiles = Array.from(e.target.files);
        setFiles(selectedFiles);
    };

    const handleSubmit = async (e) => {
        e.preventDefault();
        
        if (files.length === 0) {
            alert('Please select files to upload');
            return;
        }

        const formData = new FormData();
        files.forEach(file => {
            formData.append('files', file);
        });

        try {
            setUploading(true);
            setUploadProgress(0);

            const response = await axios.post(
                'http://localhost:8000/api/upload',
                formData,
                {
                    headers: {
                        'Content-Type': 'multipart/form-data',
                    },
                    onUploadProgress: (progressEvent) => {
                        const progress = Math.round(
                            (progressEvent.loaded * 100) / progressEvent.total
                        );
                        setUploadProgress(progress);
                    },
                }
            );

            setUploadedFiles(response.data.files);
            alert(response.data.message);
            setFiles([]);
            
        } catch (error) {
            console.error('Upload error:', error);
            alert(error.response?.data?.message || 'Upload failed');
        } finally {
            setUploading(false);
            setUploadProgress(0);
        }
    };

    return (
        <div className="max-w-md mx-auto p-6">
            <h1 className="text-2xl font-bold mb-4">File Upload</h1>
            
            <form onSubmit={handleSubmit} className="space-y-4">
                <div>
                    <input
                        type="file"
                        multiple
                        onChange={handleFileChange}
                        className="w-full p-2 border rounded"
                        accept=".jpg,.jpeg,.png,.gif,.pdf,.txt"
                    />
                </div>
                
                {files.length > 0 && (
                    <div>
                        <h3>Selected Files:</h3>
                        <ul>
                            {files.map((file, index) => (
                                <li key={index}>{file.name}</li>
                            ))}
                        </ul>
                    </div>
                )}
                
                {uploading && (
                    <div>
                        <div className="bg-gray-200 rounded">
                            <div 
                                className="bg-blue-500 h-4 rounded"
                                style={{ width: `${uploadProgress}%` }}
                            ></div>
                        </div>
                        <p>{uploadProgress}% uploaded</p>
                    </div>
                )}
                
                <button
                    type="submit"
                    disabled={uploading || files.length === 0}
                    className="w-full bg-blue-500 text-white p-2 rounded disabled:opacity-50"
                >
                    {uploading ? 'Uploading...' : 'Upload Files'}
                </button>
            </form>

            {uploadedFiles.length > 0 && (
                <div className="mt-6">
                    <h3 className="font-bold">Uploaded Files:</h3>
                    <ul>
                        {uploadedFiles.map((file, index) => (
                            <li key={index}>
                                <a 
                                    href={`http://localhost:8000${file.path}`}
                                    target="_blank"
                                    rel="noopener noreferrer"
                                    className="text-blue-500 underline"
                                >
                                    {file.originalName}
                                </a>
                                <span className="text-gray-500 ml-2">
                                    ({Math.round(file.size / 1024)}KB)
                                </span>
                            </li>
                        ))}
                    </ul>
                </div>
            )}
        </div>
    );
}
```

## Testing Your Implementation

### Running the Application
1. **Start the Express backend:**
   ```bash
   node server.js
   ```

2. **Start the Next.js frontend:**
   ```bash
   npm run dev
   ```

3. **Navigate to your file upload form in the browser**

### Test Scenarios
- ✅ **File Upload**: Upload various file types (images, PDFs, text files)
- ✅ **Progress Tracking**: Verify upload progress is displayed correctly
- ✅ **File Type Validation**: Test with invalid file types
- ✅ **Size Validation**: Test with files exceeding size limits
- ✅ **Multiple Files**: Upload multiple files simultaneously
- ✅ **Error Handling**: Verify proper error messages for various scenarios
- ✅ **File Storage**: Check that files are saved in the uploads directory
- ✅ **File Access**: Verify uploaded files can be accessed via direct URLs

### API Endpoints Testing with curl
```bash
# Upload files
curl -X POST http://localhost:8000/api/upload \
  -F "files=@/path/to/your/file1.jpg" \
  -F "files=@/path/to/your/file2.pdf"

# Get list of uploaded files
curl -X GET http://localhost:8000/api/files
```

## Key Features Implemented

### 1. Multipart Form Data Handling
- **Frontend**: FormData object for file and form field transmission
- **Backend**: Multer middleware for parsing and file extraction
- **Storage**: Configurable disk storage with unique naming

### 2. File Validation & Security
- **File Type Filtering**: Whitelist approach for allowed MIME types
- **Size Limitations**: Configurable maximum file size per upload
- **Count Limits**: Maximum number of files per request
- **Secure Naming**: Timestamp-based unique filenames

### 3. Error Handling & User Feedback
- **Frontend**: Comprehensive error catching and user notifications
- **Backend**: Custom error middleware for different failure scenarios
- **Validation**: Clear error messages for validation failures

### 4. Cross-Origin Resource Sharing (CORS)
- **Configuration**: Proper CORS setup for frontend-backend communication
- **Security**: Origin-specific access control
- **Methods**: Allowed HTTP methods specification

### 5. Upload Progress Tracking
- **Real-time Progress**: Axios onUploadProgress callback implementation
- **User Experience**: Visual progress bar and percentage display
- **Large File Handling**: Chunked upload processing

## Security Considerations
- File type validation using MIME type checking
- File size limitations to prevent abuse
- Unique filename generation to prevent conflicts
- Directory traversal protection
- CORS configuration for controlled access

## Performance Features
- Efficient file streaming with Multer
- Progress tracking for large files
- Static file serving for uploaded content
- Configurable upload limits

## Technologies Used
- **Node.js**: Runtime environment
- **Express.js**: Web framework
- **Multer**: File upload middleware
- **CORS**: Cross-origin resource sharing
- **Morgan**: HTTP request logging
- **Axios**: HTTP client for frontend requests

## Future Enhancements
- [ ] Database integration for file metadata
- [ ] User authentication and file ownership
- [ ] Cloud storage integration (AWS S3, Google Cloud)
- [ ] Image resizing and optimization
- [ ] Virus scanning for uploaded files
- [ ] File encryption for sensitive content
- [ ] Bulk file operations
- [ ] File sharing and permissions

## Troubleshooting

### Common Issues
1. **CORS Errors**: Ensure frontend URL is correctly configured in CORS settings
2. **File Size Errors**: Check MAX_FILE_SIZE environment variable
3. **Permission Errors**: Verify write permissions for uploads directory
4. **Port Conflicts**: Ensure backend and frontend use different ports

### Debug Tips
- Check server logs for detailed error information
- Use browser developer tools to inspect network requests
- Verify file permissions and directory structure
- Test API endpoints directly with curl or Postman
