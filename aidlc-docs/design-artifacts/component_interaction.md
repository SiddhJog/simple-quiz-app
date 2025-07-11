# Component Interaction Design

## Overview
This document describes the high-level component interactions, data flow, and dependencies within the Simple Quiz Application system.

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Frontend Layer (React.js)                │
├─────────────────────────────────────────────────────────────┤
│  Authentication │  Quiz Components │  Results │  Admin      │
│  Components     │                  │          │  Components │
└─────────────────┬───────────────────┬──────────┬─────────────┘
                  │                   │          │
                  │    HTTPS/REST     │          │
                  │                   │          │
┌─────────────────┴───────────────────┴──────────┴─────────────┐
│                API Gateway (AWS)                             │
└─────────────────┬───────────────────┬──────────┬─────────────┘
                  │                   │          │
┌─────────────────┴─┐ ┌───────────────┴─┐ ┌──────┴─┐ ┌─────────┐
│  Auth Service     │ │  Quiz Service   │ │ Score  │ │ Admin   │
│  (Lambda)         │ │  (Lambda)       │ │Service │ │Service  │
└─────────────────┬─┘ └───────────────┬─┘ └──────┬─┘ └─────┬───┘
                  │                   │          │         │
                  └───────────────────┼──────────┼─────────┘
                                      │          │
                  ┌───────────────────┴──────────┴─────────────┐
                  │         DynamoDB (Single Table)            │
                  └────────────────────────────────────────────┘
                                      │
                  ┌───────────────────┴─────────────────────────┐
                  │              S3 (Images)                    │
                  └─────────────────────────────────────────────┘
```

## Frontend Component Hierarchy

### React Component Structure
```
App
├── AuthProvider (Context)
├── Router
│   ├── PublicRoute
│   │   ├── LoginPage
│   │   │   └── LoginForm
│   │   └── RegisterPage
│   │       └── RegisterForm
│   └── PrivateRoute
│       ├── DashboardPage
│       │   ├── TopicSelector
│       │   └── UserStats
│       ├── QuizPage
│       │   ├── QuizHeader
│       │   ├── QuestionDisplay
│       │   ├── AnswerOptions
│       │   ├── Timer
│       │   └── ProgressBar
│       ├── ResultsPage
│       │   ├── ScoreDisplay
│       │   └── LeaderboardView
│       └── AdminPage (Admin only)
│           ├── QuestionManager
│           ├── ImageUploader
│           └── TopicManager
└── SharedComponents
    ├── Header
    ├── Navigation
    ├── ErrorBoundary
    ├── LoadingSpinner
    └── Modal
```

### Component Responsibilities

**AuthProvider**:
- Manages authentication state
- Provides user context to child components
- Handles token storage and validation

**LoginForm / RegisterForm**:
- User input validation
- API calls to authentication service
- Error handling and display

**TopicSelector**:
- Fetches available topics from API
- Handles topic selection
- Triggers quiz initialization

**QuizPage Components**:
- **QuestionDisplay**: Renders question text and images
- **AnswerOptions**: Handles answer selection
- **Timer**: Client-side 15-second countdown
- **ProgressBar**: Shows quiz progress (1-10)

**ResultsPage Components**:
- **ScoreDisplay**: Shows final score
- **LeaderboardView**: Displays rankings

**Admin Components**:
- **QuestionManager**: CRUD operations for questions
- **ImageUploader**: Handles image uploads to S3

## Backend Service Interactions

### Lambda Function Architecture
```
API Gateway
├── /auth/* → AuthService Lambda
├── /topics → QuizService Lambda
├── /quiz/* → QuizService Lambda
├── /scores/* → ScoreService Lambda
├── /leaderboard → ScoreService Lambda
└── /admin/* → AdminService Lambda
```

### Service Dependencies

**AuthService**:
- **Dependencies**: DynamoDB (User table access)
- **Responsibilities**: User registration, login, token validation
- **Data Access**: User entity CRUD operations

**QuizService**:
- **Dependencies**: DynamoDB (Topic, Question, Quiz tables), AuthService (token validation)
- **Responsibilities**: Topic listing, quiz initialization, question serving
- **Data Access**: Topic and Question entity read operations, Quiz entity CRUD

**ScoreService**:
- **Dependencies**: DynamoDB (Score, Leaderboard tables), AuthService (token validation)
- **Responsibilities**: Score calculation, persistence, leaderboard management
- **Data Access**: Score entity CRUD, Leaderboard entity updates

**AdminService**:
- **Dependencies**: DynamoDB (Question, Topic tables), S3 (image storage), AuthService (admin validation)
- **Responsibilities**: Question management, image uploads, content administration
- **Data Access**: Question entity CRUD, S3 object operations

## Data Flow Diagrams

### User Authentication Flow
```
Frontend → API Gateway → AuthService → DynamoDB
    ↓
LoginForm
    ↓
POST /auth/login
    ↓
Validate credentials
    ↓
Generate JWT token
    ↓
Return token to frontend
    ↓
Store token in localStorage
    ↓
Include in subsequent API calls
```

### Quiz Taking Flow
```
Frontend → API Gateway → QuizService → DynamoDB
    ↓
TopicSelector
    ↓
GET /topics
    ↓
Display topics
    ↓
User selects topic
    ↓
POST /quiz/start
    ↓
Select 10 random questions
    ↓
Return quiz data (without answers)
    ↓
QuizPage renders questions
    ↓
User answers questions (client-side timer)
    ↓
POST /quiz/submit
    ↓
Calculate score
    ↓
Update leaderboard
    ↓
Return results
```

### Admin Content Management Flow
```
Frontend → API Gateway → AdminService → DynamoDB/S3
    ↓
AdminPage
    ↓
POST /admin/questions
    ↓
Validate admin role
    ↓
Create question in DynamoDB
    ↓
POST /admin/images (if image)
    ↓
Upload to S3
    ↓
Update question with image URL
    ↓
Return success response
```

## Cross-Layer Interactions

### Authentication Flow Across Layers
```
1. Frontend: User submits login form
2. API Gateway: Routes to AuthService
3. AuthService: Validates credentials against DynamoDB
4. AuthService: Generates JWT token
5. Frontend: Stores token in localStorage
6. Frontend: Includes token in Authorization header
7. API Gateway: Validates token for protected routes
8. Backend Services: Receive validated user context
```

### Error Handling Propagation
```
Backend Service Error
    ↓
Generic error response (no sensitive data)
    ↓
API Gateway (logs detailed error)
    ↓
Frontend receives generic error
    ↓
ErrorBoundary catches React errors
    ↓
User-friendly error message displayed
    ↓
Error logged to console (development only)
```

### Real-time Features Implementation

**Client-Side Timer**:
```
QuizPage Component
    ↓
Timer Hook (useEffect + setInterval)
    ↓
15-second countdown per question
    ↓
Auto-advance on timeout
    ↓
No server-side timer validation
```

**Near Real-time Leaderboard**:
```
Score submission
    ↓
ScoreService updates leaderboard
    ↓
Eventually consistent updates
    ↓
Frontend polls leaderboard API
    ↓
Updates displayed rankings
```

## Component Integration Points

### Frontend-Backend Integration
```
React Components ←→ API Service Layer ←→ Backend Services
    ↓                    ↓                    ↓
State Management    HTTP Requests      Business Logic
    ↓                    ↓                    ↓
Local Storage      Request/Response    Data Persistence
```

### API Gateway Configuration
```
CORS Configuration:
- Allow Origins: Frontend domain
- Allow Methods: GET, POST, PUT, DELETE
- Allow Headers: Authorization, Content-Type

Rate Limiting:
- Per-user limits based on JWT token
- Per-IP limits for authentication endpoints

Request Validation:
- JSON schema validation
- Required header validation
- Token format validation
```

### Authentication & Authorization Integration
```
Protected Route (Frontend)
    ↓
Check localStorage for token
    ↓
If no token → Redirect to login
    ↓
If token exists → Include in API calls
    ↓
API Gateway validates token
    ↓
Extract user info from token
    ↓
Pass user context to Lambda
    ↓
Lambda checks user permissions
```

## Dependency Mapping

### Frontend Dependencies
```
React Components
├── React Router (navigation)
├── Axios/Fetch (API calls)
├── Context API (state management)
└── Custom Hooks (reusable logic)

External Dependencies:
├── Authentication state
├── API availability
└── Network connectivity
```

### Backend Dependencies
```
Lambda Functions
├── DynamoDB (data persistence)
├── S3 (image storage)
├── JWT library (token handling)
└── Validation libraries

Service Dependencies:
├── AuthService ← All other services
├── QuizService ← ScoreService
└── AdminService ← Independent
```

### Database Dependencies
```
DynamoDB Single Table
├── User entities
├── Question entities
├── Quiz entities
├── Score entities
└── Leaderboard entities

Access Patterns:
├── User by email (login)
├── Questions by topic (quiz generation)
├── Scores by user (history)
└── Leaderboard by topic (rankings)
```

## Error Handling Strategy

### Frontend Error Handling
```
Component Level:
├── Try-catch blocks for async operations
├── Error states in component state
├── User-friendly error messages
└── Fallback UI components

Application Level:
├── ErrorBoundary for React errors
├── Global error handler for API calls
├── Network error detection
└── Retry mechanisms for failed requests
```

### Backend Error Handling
```
Service Level:
├── Input validation errors
├── Business logic errors
├── Database operation errors
└── External service errors

System Level:
├── Lambda timeout handling
├── DynamoDB throttling
├── S3 upload failures
└── API Gateway errors
```

### Error Logging Strategy
```
Frontend:
├── Console logging (development)
├── Error reporting service (production)
└── User action tracking

Backend:
├── CloudWatch logs for Lambda
├── Structured logging with context
├── Error metrics and alarms
└── Audit trail for admin actions
```

## Performance Considerations

### Frontend Optimization
```
Component Optimization:
├── React.memo for expensive components
├── useMemo for expensive calculations
├── useCallback for event handlers
└── Lazy loading for admin components

Network Optimization:
├── Request caching
├── Image optimization
├── Bundle splitting
└── CDN for static assets
```

### Backend Optimization
```
Lambda Optimization:
├── Connection pooling for DynamoDB
├── Warm-up strategies
├── Memory allocation tuning
└── Cold start mitigation

Database Optimization:
├── Efficient access patterns
├── Proper indexing strategy
├── Batch operations where possible
└── Connection reuse
```

## Monitoring & Observability

### Component Health Monitoring
```
Frontend:
├── Page load times
├── API response times
├── Error rates by component
└── User interaction metrics

Backend:
├── Lambda execution metrics
├── DynamoDB performance metrics
├── API Gateway metrics
└── Error rates by service
```

### Integration Monitoring
```
Cross-Service Communication:
├── API call success rates
├── Authentication failure rates
├── Database connection health
└── S3 upload success rates

End-to-End Monitoring:
├── User journey completion rates
├── Quiz completion rates
├── Score submission success
└── Leaderboard update latency
```

---

*Document Version: 1.0*
*Created: [Current Date]*
*Status: Complete*
*Interaction Level: High-Level Overview*
