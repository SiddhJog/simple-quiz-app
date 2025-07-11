# API Specification

## Overview
This document provides the complete REST API specification for the Simple Quiz Application using OpenAPI 3.0 standards.

## Base Configuration

**Base URL**: `/api/v1`  
**Protocol**: HTTPS  
**Content-Type**: `application/json`  
**Authentication**: Bearer Token (JWT)

## Authentication APIs

### POST /api/v1/auth/register
**Description**: Register a new user account

**Request Schema**:
```json
{
  "email": "string (required, email format)",
  "password": "string (required, min 8 chars, 1 uppercase, 1 lowercase, 1 special)"
}
```

**Response Schema**:
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "userId": "string",
    "email": "string"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Invalid input data
- `409 Conflict`: Email already exists
- `500 Internal Server Error`: Server error

**Example Request**:
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Example Response**:
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "userId": "usr_1234567890",
    "email": "user@example.com"
  }
}
```

---

### POST /api/v1/auth/login
**Description**: Authenticate user and return JWT token

**Request Schema**:
```json
{
  "email": "string (required)",
  "password": "string (required)"
}
```

**Response Schema**:
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "token": "string (JWT)",
    "userId": "string",
    "expiresIn": "number (seconds)"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Missing credentials
- `401 Unauthorized`: Invalid credentials
- `500 Internal Server Error`: Server error

**Example Request**:
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Example Response**:
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "userId": "usr_1234567890",
    "expiresIn": 3600
  }
}
```

---

### GET /api/v1/auth/validate
**Description**: Validate JWT token and return user info

**Headers**:
```
Authorization: Bearer <jwt_token>
```

**Response Schema**:
```json
{
  "success": true,
  "message": "Token is valid",
  "data": {
    "userId": "string",
    "email": "string",
    "role": "string"
  }
}
```

**Error Responses**:
- `401 Unauthorized`: Invalid or expired token
- `500 Internal Server Error`: Server error

---

### POST /api/v1/auth/logout
**Description**: Logout user (client-side token removal)

**Headers**:
```
Authorization: Bearer <jwt_token>
```

**Response Schema**:
```json
{
  "success": true,
  "message": "Logout successful"
}
```

## Quiz Management APIs

### GET /api/v1/topics
**Description**: Get all available quiz topics

**Headers**:
```
Authorization: Bearer <jwt_token>
```

**Response Schema**:
```json
{
  "success": true,
  "message": "Topics retrieved successfully",
  "data": {
    "topics": [
      {
        "id": "string",
        "name": "string",
        "description": "string",
        "questionCount": "number",
        "isActive": "boolean"
      }
    ]
  }
}
```

**Error Responses**:
- `401 Unauthorized`: Invalid token
- `500 Internal Server Error`: Server error

**Example Response**:
```json
{
  "success": true,
  "message": "Topics retrieved successfully",
  "data": {
    "topics": [
      {
        "id": "topic_general",
        "name": "General Knowledge",
        "description": "General knowledge questions",
        "questionCount": 25,
        "isActive": true
      },
      {
        "id": "topic_physics",
        "name": "Physics",
        "description": "Physics questions",
        "questionCount": 30,
        "isActive": true
      }
    ]
  }
}
```

---

### POST /api/v1/quiz/start
**Description**: Initialize a new quiz session

**Headers**:
```
Authorization: Bearer <jwt_token>
```

**Request Schema**:
```json
{
  "topicId": "string (required)"
}
```

**Response Schema**:
```json
{
  "success": true,
  "message": "Quiz started successfully",
  "data": {
    "quizId": "string",
    "topicId": "string",
    "totalQuestions": 10,
    "timePerQuestion": 15,
    "questions": [
      {
        "id": "string",
        "text": "string",
        "options": ["string", "string", "string", "string"],
        "imageUrl": "string (optional)"
      }
    ]
  }
}
```

**Error Responses**:
- `400 Bad Request`: Invalid topic ID
- `401 Unauthorized`: Invalid token
- `409 Conflict`: User already has active quiz
- `500 Internal Server Error`: Server error

**Example Request**:
```json
{
  "topicId": "topic_general"
}
```

**Example Response**:
```json
{
  "success": true,
  "message": "Quiz started successfully",
  "data": {
    "quizId": "quiz_1234567890",
    "topicId": "topic_general",
    "totalQuestions": 10,
    "timePerQuestion": 15,
    "questions": [
      {
        "id": "q1",
        "text": "What is the capital of France?",
        "options": ["London", "Berlin", "Paris", "Madrid"],
        "imageUrl": null
      }
    ]
  }
}
```

---

### POST /api/v1/quiz/submit
**Description**: Submit quiz answers and get results

**Headers**:
```
Authorization: Bearer <jwt_token>
```

**Request Schema**:
```json
{
  "quizId": "string (required)",
  "answers": [
    {
      "questionId": "string",
      "selectedOption": "number (0-3, null for no answer)"
    }
  ]
}
```

**Response Schema**:
```json
{
  "success": true,
  "message": "Quiz submitted successfully",
  "data": {
    "score": "number (0-10)",
    "totalQuestions": 10,
    "correctAnswers": "number",
    "leaderboardPosition": "number",
    "completedAt": "string (ISO datetime)"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Invalid quiz ID or answers
- `401 Unauthorized`: Invalid token
- `404 Not Found`: Quiz not found
- `409 Conflict`: Quiz already submitted
- `500 Internal Server Error`: Server error

**Example Request**:
```json
{
  "quizId": "quiz_1234567890",
  "answers": [
    {
      "questionId": "q1",
      "selectedOption": 2
    },
    {
      "questionId": "q2",
      "selectedOption": null
    }
  ]
}
```

**Example Response**:
```json
{
  "success": true,
  "message": "Quiz submitted successfully",
  "data": {
    "score": 7,
    "totalQuestions": 10,
    "correctAnswers": 7,
    "leaderboardPosition": 15,
    "completedAt": "2024-01-15T10:30:00Z"
  }
}
```

## Score & Leaderboard APIs

### GET /api/v1/scores/user/{userId}
**Description**: Get user's quiz score history

**Headers**:
```
Authorization: Bearer <jwt_token>
```

**Path Parameters**:
- `userId`: string (required)

**Query Parameters**:
- `topicId`: string (optional, filter by topic)
- `limit`: number (optional, default 10)
- `offset`: number (optional, default 0)

**Response Schema**:
```json
{
  "success": true,
  "message": "Scores retrieved successfully",
  "data": {
    "scores": [
      {
        "scoreId": "string",
        "topicId": "string",
        "topicName": "string",
        "score": "number",
        "totalQuestions": 10,
        "completedAt": "string (ISO datetime)"
      }
    ],
    "pagination": {
      "total": "number",
      "limit": "number",
      "offset": "number"
    }
  }
}
```

**Error Responses**:
- `401 Unauthorized`: Invalid token
- `403 Forbidden`: Cannot access other user's scores
- `500 Internal Server Error`: Server error

---

### GET /api/v1/leaderboard
**Description**: Get leaderboard rankings

**Headers**:
```
Authorization: Bearer <jwt_token>
```

**Query Parameters**:
- `topicId`: string (optional, filter by topic)
- `limit`: number (optional, default 10)

**Response Schema**:
```json
{
  "success": true,
  "message": "Leaderboard retrieved successfully",
  "data": {
    "rankings": [
      {
        "rank": "number",
        "userId": "string",
        "userName": "string",
        "score": "number",
        "achievedAt": "string (ISO datetime)"
      }
    ],
    "userRanking": {
      "rank": "number",
      "score": "number",
      "achievedAt": "string (ISO datetime)"
    }
  }
}
```

**Error Responses**:
- `401 Unauthorized`: Invalid token
- `500 Internal Server Error`: Server error

**Example Response**:
```json
{
  "success": true,
  "message": "Leaderboard retrieved successfully",
  "data": {
    "rankings": [
      {
        "rank": 1,
        "userId": "usr_9876543210",
        "userName": "TopPlayer",
        "score": 10,
        "achievedAt": "2024-01-15T09:00:00Z"
      },
      {
        "rank": 2,
        "userId": "usr_1111111111",
        "userName": "SecondPlace",
        "score": 9,
        "achievedAt": "2024-01-15T10:00:00Z"
      }
    ],
    "userRanking": {
      "rank": 15,
      "score": 7,
      "achievedAt": "2024-01-15T10:30:00Z"
    }
  }
}
```

## Admin Management APIs

### POST /api/v1/admin/questions
**Description**: Create a new question (Admin only)

**Headers**:
```
Authorization: Bearer <jwt_token>
```

**Request Schema**:
```json
{
  "topicId": "string (required)",
  "questionText": "string (required)",
  "options": ["string", "string", "string", "string"] (required, exactly 4),
  "correctAnswerIndex": "number (required, 0-3)",
  "imageUrl": "string (optional)"
}
```

**Response Schema**:
```json
{
  "success": true,
  "message": "Question created successfully",
  "data": {
    "questionId": "string",
    "topicId": "string",
    "questionText": "string",
    "options": ["string", "string", "string", "string"],
    "correctAnswerIndex": "number",
    "imageUrl": "string",
    "createdAt": "string (ISO datetime)"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Invalid question data
- `401 Unauthorized`: Invalid token
- `403 Forbidden`: Not admin user
- `500 Internal Server Error`: Server error

---

### PUT /api/v1/admin/questions/{questionId}
**Description**: Update existing question (Admin only)

**Headers**:
```
Authorization: Bearer <jwt_token>
```

**Path Parameters**:
- `questionId`: string (required)

**Request Schema**:
```json
{
  "questionText": "string (optional)",
  "options": ["string", "string", "string", "string"] (optional),
  "correctAnswerIndex": "number (optional, 0-3)",
  "imageUrl": "string (optional)"
}
```

**Response Schema**:
```json
{
  "success": true,
  "message": "Question updated successfully",
  "data": {
    "questionId": "string",
    "topicId": "string",
    "questionText": "string",
    "options": ["string", "string", "string", "string"],
    "correctAnswerIndex": "number",
    "imageUrl": "string",
    "updatedAt": "string (ISO datetime)"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Invalid question data
- `401 Unauthorized`: Invalid token
- `403 Forbidden`: Not admin user
- `404 Not Found`: Question not found
- `500 Internal Server Error`: Server error

---

### DELETE /api/v1/admin/questions/{questionId}
**Description**: Delete question (Admin only)

**Headers**:
```
Authorization: Bearer <jwt_token>
```

**Path Parameters**:
- `questionId`: string (required)

**Response Schema**:
```json
{
  "success": true,
  "message": "Question deleted successfully"
}
```

**Error Responses**:
- `401 Unauthorized`: Invalid token
- `403 Forbidden`: Not admin user
- `404 Not Found`: Question not found
- `409 Conflict`: Question in use by active quizzes
- `500 Internal Server Error`: Server error

---

### POST /api/v1/admin/images
**Description**: Upload question image (Admin only)

**Headers**:
```
Authorization: Bearer <jwt_token>
Content-Type: multipart/form-data
```

**Request Body**:
```
image: file (required, JPG/PNG, max 1MB)
questionId: string (required)
```

**Response Schema**:
```json
{
  "success": true,
  "message": "Image uploaded successfully",
  "data": {
    "imageUrl": "string (S3 URL)",
    "questionId": "string"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Invalid file format or size
- `401 Unauthorized`: Invalid token
- `403 Forbidden`: Not admin user
- `404 Not Found`: Question not found
- `500 Internal Server Error`: Server error

---

### GET /api/v1/admin/topics/{topicId}/questions
**Description**: Get all questions for a topic (Admin only)

**Headers**:
```
Authorization: Bearer <jwt_token>
```

**Path Parameters**:
- `topicId`: string (required)

**Query Parameters**:
- `limit`: number (optional, default 50)
- `offset`: number (optional, default 0)

**Response Schema**:
```json
{
  "success": true,
  "message": "Questions retrieved successfully",
  "data": {
    "questions": [
      {
        "questionId": "string",
        "questionText": "string",
        "options": ["string", "string", "string", "string"],
        "correctAnswerIndex": "number",
        "imageUrl": "string",
        "isActive": "boolean",
        "createdAt": "string (ISO datetime)"
      }
    ],
    "pagination": {
      "total": "number",
      "limit": "number",
      "offset": "number"
    }
  }
}
```

**Error Responses**:
- `401 Unauthorized`: Invalid token
- `403 Forbidden`: Not admin user
- `404 Not Found`: Topic not found
- `500 Internal Server Error`: Server error

## Error Response Format

All API errors follow a consistent format:

```json
{
  "success": false,
  "error": {
    "code": "string (error code)",
    "message": "string (human readable message)",
    "details": "string (optional additional details)"
  }
}
```

### Common Error Codes

**Authentication Errors**:
- `AUTH_INVALID_CREDENTIALS`: Invalid email or password
- `AUTH_TOKEN_EXPIRED`: JWT token has expired
- `AUTH_TOKEN_INVALID`: JWT token is malformed or invalid
- `AUTH_EMAIL_EXISTS`: Email already registered

**Validation Errors**:
- `VALIDATION_FAILED`: Request data validation failed
- `VALIDATION_EMAIL_FORMAT`: Invalid email format
- `VALIDATION_PASSWORD_POLICY`: Password doesn't meet policy

**Business Logic Errors**:
- `QUIZ_ALREADY_ACTIVE`: User has an active quiz session
- `QUIZ_NOT_FOUND`: Quiz session not found
- `QUIZ_ALREADY_COMPLETED`: Quiz already submitted
- `TOPIC_INSUFFICIENT_QUESTIONS`: Topic has less than 10 questions

**Authorization Errors**:
- `ACCESS_DENIED`: User doesn't have required permissions
- `ADMIN_REQUIRED`: Admin role required for this operation

**Resource Errors**:
- `RESOURCE_NOT_FOUND`: Requested resource not found
- `RESOURCE_CONFLICT`: Resource conflict (e.g., duplicate)

**Server Errors**:
- `INTERNAL_ERROR`: Generic server error
- `DATABASE_ERROR`: Database operation failed
- `EXTERNAL_SERVICE_ERROR`: External service (S3) error

## Rate Limiting

**Limits**:
- Authentication endpoints: 5 requests per minute per IP
- Quiz endpoints: 10 requests per minute per user
- Admin endpoints: 20 requests per minute per admin
- General endpoints: 100 requests per minute per user

**Headers**:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
```

## Authentication Flow

1. **Registration**: `POST /api/v1/auth/register`
2. **Login**: `POST /api/v1/auth/login` → Returns JWT token
3. **API Calls**: Include `Authorization: Bearer <token>` header
4. **Token Validation**: Automatic validation on protected endpoints
5. **Logout**: `POST /api/v1/auth/logout` (client removes token)

## Environment Configuration

**Development**:
```
BASE_URL: https://dev-api.quizapp.com/api/v1
JWT_EXPIRY: 1 hour
RATE_LIMIT: Relaxed
```

**Staging**:
```
BASE_URL: https://staging-api.quizapp.com/api/v1
JWT_EXPIRY: 1 hour
RATE_LIMIT: Production-like
```

**Production**:
```
BASE_URL: https://api.quizapp.com/api/v1
JWT_EXPIRY: 1 hour
RATE_LIMIT: Strict
```

---

*Document Version: 1.0*
*Created: [Current Date]*
*API Standard: OpenAPI 3.0*
*Status: Complete*
