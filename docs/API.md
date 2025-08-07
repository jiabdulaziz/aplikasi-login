# API Documentation

## Overview
This document provides comprehensive documentation for all public APIs in the Login Application (aplikasi-login).

## Base URL
```
https://api.aplikasi-login.com/v1
```

## Authentication
All API endpoints require authentication via Bearer token unless otherwise specified.

```http
Authorization: Bearer <your-token>
```

## Endpoints

### Authentication Endpoints

#### POST /auth/login
Authenticates a user with email and password.

**Request:**
```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "12345",
      "email": "user@example.com",
      "name": "John Doe",
      "createdAt": "2024-01-01T00:00:00Z"
    },
    "expiresIn": 3600
  },
  "message": "Login successful"
}
```

**Error Response:**
```json
{
  "success": false,
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Email atau password tidak valid"
  }
}
```

**Status Codes:**
- `200` - Login successful
- `400` - Bad request (validation errors)
- `401` - Invalid credentials
- `429` - Too many requests

#### POST /auth/logout
Logs out the current user and invalidates the token.

**Request:**
```http
POST /auth/logout
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "message": "Logout successful"
}
```

#### POST /auth/refresh
Refreshes the authentication token.

**Request:**
```http
POST /auth/refresh
Authorization: Bearer <refresh-token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "new-jwt-token",
    "expiresIn": 3600
  }
}
```

### User Management Endpoints

#### GET /users/profile
Retrieves the current user's profile information.

**Request:**
```http
GET /users/profile
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "12345",
    "email": "user@example.com",
    "name": "John Doe",
    "avatar": "https://example.com/avatar.jpg",
    "createdAt": "2024-01-01T00:00:00Z",
    "lastLogin": "2024-01-15T10:30:00Z"
  }
}
```

#### PUT /users/profile
Updates the current user's profile information.

**Request:**
```http
PUT /users/profile
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "Jane Doe",
  "avatar": "https://example.com/new-avatar.jpg"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "12345",
    "email": "user@example.com",
    "name": "Jane Doe",
    "avatar": "https://example.com/new-avatar.jpg",
    "updatedAt": "2024-01-15T11:00:00Z"
  },
  "message": "Profile updated successfully"
}
```

## Error Handling

### Standard Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": {
      "field": "specific error details"
    }
  }
}
```

### Common Error Codes
- `VALIDATION_ERROR` - Request validation failed
- `UNAUTHORIZED` - Authentication required
- `FORBIDDEN` - Insufficient permissions
- `NOT_FOUND` - Resource not found
- `RATE_LIMIT_EXCEEDED` - Too many requests
- `INTERNAL_ERROR` - Server error

## Rate Limiting
API requests are limited to:
- 100 requests per minute for authenticated users
- 20 requests per minute for unauthenticated endpoints

Rate limit headers are included in responses:
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1642694400
```

## Request/Response Examples

### JavaScript/Fetch Example
```javascript
// Login example
const login = async (email, password) => {
  try {
    const response = await fetch('/auth/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ email, password })
    });
    
    const data = await response.json();
    
    if (data.success) {
      localStorage.setItem('token', data.data.token);
      return data.data.user;
    } else {
      throw new Error(data.error.message);
    }
  } catch (error) {
    console.error('Login failed:', error);
    throw error;
  }
};
```

### cURL Examples
```bash
# Login
curl -X POST https://api.aplikasi-login.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"password123"}'

# Get profile
curl -X GET https://api.aplikasi-login.com/v1/users/profile \
  -H "Authorization: Bearer your-token-here"
```

## SDK Examples

### JavaScript SDK
```javascript
import { LoginAPI } from 'aplikasi-login-sdk';

const api = new LoginAPI({
  baseURL: 'https://api.aplikasi-login.com/v1',
  apiKey: 'your-api-key'
});

// Login
const user = await api.auth.login('user@example.com', 'password');

// Get profile
const profile = await api.users.getProfile();
```

## Webhooks
Configure webhooks to receive real-time notifications for user events.

### Supported Events
- `user.login` - User successfully logged in
- `user.logout` - User logged out
- `user.profile.updated` - User profile was updated

### Webhook Payload Example
```json
{
  "event": "user.login",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "userId": "12345",
    "email": "user@example.com",
    "ipAddress": "192.168.1.1",
    "userAgent": "Mozilla/5.0..."
  }
}
```