# API Specification

## Overview
This document defines the REST API endpoints for the quiz application backend, including request/response schemas, authentication, and error handling.

## Base Configuration
- **Base URL**: `http://localhost:8000/api/v1`
- **Protocol**: HTTP/HTTPS
- **Data Format**: JSON
- **Authentication**: Session cookies
- **CORS**: Enabled for frontend domain

## Authentication

### Session Management
- **Method**: Session cookies
- **Cookie Name**: `quiz_session`
- **Expiration**: Browser session (no persistence)
- **Security**: HttpOnly, Secure (in production)

### Authentication Flow
1. User validates ID → Server creates session cookie
2. All subsequent requests include session cookie
3. Server validates session for protected endpoints
4. Session expires when browser closes

## API Endpoints

### 1. User Authentication

#### POST /auth/validate-user
**Purpose**: Validate user ID and create session

**Request**:
```json
{
  "user_id": "string (alphanumeric, 3-20 chars)"
}
```

**Response (Success - 200)**:
```json
{
  "success": true,
  "message": "User validated successfully",
  "user_id": "john123",
  "session_created": true
}
```

**Response (Error - 400)**:
```json
{
  "success": false,
  "error": "DUPLICATE_USER_ID",
  "message": "User ID already exists. Please choose a different ID."
}
```

**Response (Error - 400)**:
```json
{
  "success": false,
  "error": "INVALID_FORMAT",
  "message": "User ID must be alphanumeric and 3-20 characters long."
}
```

#### POST /auth/logout
**Purpose**: Clear user session

**Request**: No body required

**Response (Success - 200)**:
```json
{
  "success": true,
  "message": "Session cleared successfully"
}
```

### 2. Dashboard Data

#### GET /dashboard/stats
**Purpose**: Get user dashboard statistics
**Authentication**: Required

**Response (Success - 200)**:
```json
{
  "success": true,
  "data": {
    "user_id": "john123",
    "recent_scores": [
      {
        "score": 8,
        "total": 10,
        "topic": "General Knowledge",
        "completion_time": "2024-01-15T10:30:00Z"
      }
    ],
    "overall_average": 7.2,
    "total_quizzes": 15,
    "average_response_time": 12.5,
    "topic_performance": [
      {
        "topic": "General Knowledge",
        "average_score": 7.5,
        "quizzes_taken": 8
      },
      {
        "topic": "Science",
        "average_score": 6.8,
        "quizzes_taken": 4
      }
    ]
  }
}
```

**Response (Error - 401)**:
```json
{
  "success": false,
  "error": "UNAUTHORIZED",
  "message": "Valid session required"
}
```

### 3. Quiz Topics

#### GET /quiz/topics
**Purpose**: Get available quiz topics
**Authentication**: Required

**Response (Success - 200)**:
```json
{
  "success": true,
  "data": {
    "topics": [
      {
        "topic_id": 1,
        "topic_name": "General Knowledge",
        "description": "Questions covering various general topics",
        "question_count": 150
      },
      {
        "topic_id": 2,
        "topic_name": "Science",
        "description": "Questions about scientific concepts and discoveries",
        "question_count": 120
      },
      {
        "topic_id": 3,
        "topic_name": "Films",
        "description": "Questions about movies, actors, and cinema",
        "question_count": 110
      }
    ]
  }
}
```

### 4. Quiz Session Management

#### POST /quiz/start
**Purpose**: Start a new quiz session
**Authentication**: Required

**Request**:
```json
{
  "topic_id": 1
}
```

**Response (Success - 200)**:
```json
{
  "success": true,
  "data": {
    "session_id": "sess_abc123",
    "topic": {
      "topic_id": 1,
      "topic_name": "General Knowledge"
    },
    "total_questions": 10,
    "time_per_question": 20
  }
}
```

#### GET /quiz/question/{session_id}/{question_number}
**Purpose**: Get specific question for quiz session
**Authentication**: Required

**Parameters**:
- `session_id`: Quiz session identifier
- `question_number`: Question number (1-10)

**Response (Success - 200)**:
```json
{
  "success": true,
  "data": {
    "question_number": 1,
    "total_questions": 10,
    "question_id": 456,
    "question_text": "What is the capital of France?",
    "options": [
      {
        "option_id": "A",
        "text": "London"
      },
      {
        "option_id": "B", 
        "text": "Berlin"
      },
      {
        "option_id": "C",
        "text": "Paris"
      },
      {
        "option_id": "D",
        "text": "Madrid"
      }
    ]
  }
}
```

**Response (Error - 404)**:
```json
{
  "success": false,
  "error": "SESSION_NOT_FOUND",
  "message": "Quiz session not found or expired"
}
```

#### POST /quiz/answer
**Purpose**: Submit answer for current question
**Authentication**: Required

**Request**:
```json
{
  "session_id": "sess_abc123",
  "question_id": 456,
  "selected_answer": "C",
  "response_time": 15.2
}
```

**Response (Success - 200)**:
```json
{
  "success": true,
  "data": {
    "correct": true,
    "question_number": 1,
    "next_question": 2,
    "quiz_completed": false
  }
}
```

**Response (Quiz Completed - 200)**:
```json
{
  "success": true,
  "data": {
    "correct": false,
    "question_number": 10,
    "quiz_completed": true,
    "final_score": 7,
    "total_questions": 10
  }
}
```

### 5. Quiz Results

#### GET /quiz/results/{session_id}
**Purpose**: Get detailed quiz results
**Authentication**: Required

**Response (Success - 200)**:
```json
{
  "success": true,
  "data": {
    "session_id": "sess_abc123",
    "user_id": "john123",
    "topic": "General Knowledge",
    "final_score": 7,
    "total_questions": 10,
    "completion_time": "2024-01-15T10:45:00Z",
    "time_taken": 180,
    "average_response_time": 18.0,
    "questions": [
      {
        "question_number": 1,
        "question_text": "What is the capital of France?",
        "user_answer": "C",
        "correct_answer": "C",
        "is_correct": true,
        "response_time": 15.2
      },
      {
        "question_number": 2,
        "question_text": "Who painted the Mona Lisa?",
        "user_answer": "B",
        "correct_answer": "A",
        "is_correct": false,
        "response_time": 20.0
      }
    ]
  }
}
```

#### POST /quiz/save-result
**Purpose**: Save quiz result to user history
**Authentication**: Required

**Request**:
```json
{
  "session_id": "sess_abc123",
  "final_score": 7,
  "completion_time": "2024-01-15T10:45:00Z",
  "time_taken": 180,
  "average_response_time": 18.0
}
```

**Response (Success - 200)**:
```json
{
  "success": true,
  "message": "Quiz result saved successfully",
  "result_id": 789
}
```

### 6. Error Handling

#### Common Error Responses

**400 Bad Request**:
```json
{
  "success": false,
  "error": "VALIDATION_ERROR",
  "message": "Invalid request data",
  "details": {
    "field": "user_id",
    "issue": "Must be alphanumeric"
  }
}
```

**401 Unauthorized**:
```json
{
  "success": false,
  "error": "UNAUTHORIZED",
  "message": "Valid session required"
}
```

**404 Not Found**:
```json
{
  "success": false,
  "error": "RESOURCE_NOT_FOUND",
  "message": "Requested resource not found"
}
```

**500 Internal Server Error**:
```json
{
  "success": false,
  "error": "INTERNAL_ERROR",
  "message": "An unexpected error occurred"
}
```

**503 Service Unavailable**:
```json
{
  "success": false,
  "error": "SERVICE_UNAVAILABLE",
  "message": "Service temporarily unavailable"
}
```

## Error Codes Reference

### Authentication Errors
- `DUPLICATE_USER_ID`: User ID already exists
- `INVALID_FORMAT`: User ID format validation failed
- `UNAUTHORIZED`: No valid session found
- `SESSION_EXPIRED`: Session has expired

### Quiz Errors
- `SESSION_NOT_FOUND`: Quiz session not found
- `INVALID_QUESTION`: Question number out of range
- `QUIZ_COMPLETED`: Attempting to answer completed quiz
- `INVALID_ANSWER`: Answer format validation failed

### Data Errors
- `VALIDATION_ERROR`: Request data validation failed
- `RESOURCE_NOT_FOUND`: Requested resource doesn't exist
- `DATABASE_ERROR`: Database operation failed

### System Errors
- `INTERNAL_ERROR`: Unexpected server error
- `SERVICE_UNAVAILABLE`: Service temporarily down
- `RATE_LIMITED`: Too many requests (future enhancement)

## Request/Response Headers

### Standard Headers
```
Content-Type: application/json
Accept: application/json
```

### Session Headers
```
Cookie: quiz_session=<session_token>
Set-Cookie: quiz_session=<session_token>; HttpOnly; Secure; SameSite=Strict
```

### CORS Headers
```
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

## Rate Limiting (Future Enhancement)
- **Limit**: 100 requests per minute per session
- **Headers**: `X-RateLimit-Limit`, `X-RateLimit-Remaining`
- **Error**: 429 Too Many Requests

## API Versioning
- **Current Version**: v1
- **URL Pattern**: `/api/v1/...`
- **Header**: `API-Version: 1.0`

## Security Considerations

### Input Validation
- All inputs validated server-side
- XSS prevention through input sanitization
- SQL injection prevention through parameterized queries
- File upload restrictions (not applicable in Phase 1)

### Session Security
- HttpOnly cookies prevent XSS access
- Secure flag for HTTPS connections
- SameSite attribute prevents CSRF
- Session timeout on browser close

### Data Protection
- No sensitive data in API responses
- Quiz answers validated server-side
- Question bank protected from unauthorized access
- User data isolated by session

## Performance Considerations

### Response Times
- Authentication: < 200ms
- Dashboard data: < 500ms
- Question retrieval: < 200ms
- Answer submission: < 300ms

### Caching
- Static topic data cached
- Question randomization performed server-side
- User statistics cached for dashboard performance

### Database Optimization
- Indexed queries for user data
- Efficient random question selection
- Connection pooling for concurrent users
