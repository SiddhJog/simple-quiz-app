# Simple Quiz Application - System Architecture

## 1. System Overview

### 1.1 Context
The Simple Quiz Application is a web-based platform that allows users to take multiple-choice quizzes on various topics. The system supports user authentication, quiz management, scoring, and leaderboards with the capability to handle 1000 concurrent users.

### 1.2 High-Level Architecture
The system follows a **serverless microservices architecture** on AWS, consisting of:
- **Frontend**: React.js Single Page Application (SPA)
- **Backend**: AWS Lambda functions with Python
- **Database**: DynamoDB single-table design
- **Storage**: S3 for question images
- **Authentication**: Custom JWT-based authentication

## 2. System Components

### 2.1 Frontend Layer (React.js)
**Technology**: React.js with modern hooks and functional components

**Key Components**:
- **Authentication Components**: Login, Registration, Session Management
- **Quiz Components**: Topic Selection, Question Display, Timer, Answer Selection
- **Results Components**: Score Display, Leaderboard
- **Admin Components**: Question Management, Image Upload
- **Shared Components**: Navigation, Error Handling, Loading States

**Responsibilities**:
- User interface rendering and interaction
- Client-side timer implementation (15-second countdown)
- Form validation and input handling
- State management for quiz session
- API communication with backend services

### 2.2 Backend Layer (AWS Lambda + Python)
**Technology**: Python 3.9+ with AWS Lambda

**API Services**:

#### Authentication Service
- `POST /auth/register` - User registration
- `POST /auth/login` - User login
- `POST /auth/logout` - User logout
- `GET /auth/validate` - Token validation

#### Quiz Service
- `GET /topics` - Get available quiz topics
- `POST /quiz/start` - Initialize quiz session
- `GET /quiz/question/{questionId}` - Get question details
- `POST /quiz/submit` - Submit quiz answers

#### Score Service
- `POST /scores` - Save quiz score
- `GET /scores/user/{userId}` - Get user score history
- `GET /leaderboard` - Get leaderboard rankings

#### Admin Service
- `POST /admin/questions` - Create question
- `PUT /admin/questions/{questionId}` - Update question
- `DELETE /admin/questions/{questionId}` - Delete question
- `POST /admin/images` - Upload question image

### 2.3 Database Layer (DynamoDB)
**Technology**: AWS DynamoDB with single-table design

**Table Schema**: `QuizAppTable`
```
Primary Key: PK (Partition Key), SK (Sort Key)
Attributes: EntityType, Data, GSI1PK, GSI1SK, TTL
```

**Entity Patterns**:
- Users: `PK=USER#{userId}`, `SK=PROFILE`
- Questions: `PK=TOPIC#{topicName}`, `SK=QUESTION#{questionId}`
- Scores: `PK=USER#{userId}`, `SK=SCORE#{timestamp}`
- Leaderboard: `PK=LEADERBOARD`, `SK=SCORE#{score}#{userId}`

**Global Secondary Indexes**:
- GSI1: `GSI1PK`, `GSI1SK` for leaderboard queries

### 2.4 Storage Layer (S3)
**Technology**: AWS S3

**Bucket Structure**:
```
quiz-app-images/
├── questions/
│   ├── {questionId}.jpg
│   └── {questionId}.png
└── temp/
    └── uploads/
```

**Configuration**:
- Maximum file size: 1MB
- Supported formats: JPG, PNG
- Public read access for question images
- Lifecycle policy for temp uploads cleanup

## 3. System Interactions & Data Flow

### 3.1 User Authentication Flow
```
1. User submits registration/login form
2. Frontend validates input (password policy)
3. API Gateway routes to Auth Lambda
4. Lambda validates credentials against DynamoDB
5. JWT token generated and returned
6. Frontend stores token for subsequent requests
7. Token included in Authorization header for protected APIs
```

### 3.2 Quiz Taking Flow
```
1. User selects topic from available options
2. Frontend calls Quiz Service to initialize quiz
3. Lambda selects 10 random questions from topic
4. Questions returned to frontend (without correct answers)
5. Frontend displays questions one by one with 15s timer
6. User selections stored in frontend state
7. On completion, answers submitted to Score Service
8. Lambda calculates score and updates leaderboard
9. Results displayed to user
```

### 3.3 Admin Content Management Flow
```
1. Admin authenticates (same system as users)
2. Admin interface loads question management
3. CRUD operations route to Admin Service
4. Image uploads go to S3 via pre-signed URLs
5. Question data stored in DynamoDB
6. Changes immediately available for quizzes
```

## 4. Technical Decisions & Rationale

### 4.1 Architecture Pattern: Serverless Microservices
**Decision**: AWS Lambda-based serverless architecture
**Rationale**:
- Auto-scaling to handle 1000 concurrent users
- Pay-per-use cost model
- No server management overhead
- Built-in high availability

### 4.2 Database: Single-Table DynamoDB Design
**Decision**: Single DynamoDB table with composite keys
**Rationale**:
- Optimized for access patterns
- Cost-effective for read-heavy workloads
- Eventual consistency aligns with requirements
- Simplified backup and maintenance

### 4.3 Authentication: Custom JWT Implementation
**Decision**: Custom JWT-based authentication
**Rationale**:
- Phase 1 requirement (AWS Cognito for Phase 2)
- Full control over user data and policies
- Stateless API design
- Enhanced password policy support

### 4.4 Timer: Client-Side Implementation
**Decision**: JavaScript-based countdown timer
**Rationale**:
- Reduces server load
- Better user experience (real-time updates)
- Network disconnection handling (quiz reset)
- Simpler implementation

### 4.5 API Design: RESTful Services
**Decision**: REST APIs with JSON payloads
**Rationale**:
- Industry standard and well-understood
- Good tooling and documentation support
- Stateless design for scalability
- Easy versioning with new endpoints

## 5. Security Architecture

### 5.1 Authentication & Authorization
- **Password Policy**: 8+ characters, 1 uppercase, 1 lowercase, 1 special character
- **JWT Tokens**: HS256 algorithm with configurable expiration
- **Session Management**: Stateless tokens, client-side storage
- **Admin Access**: Same authentication system, role-based access

### 5.2 Data Protection
- **Encryption**: DynamoDB encryption at rest
- **HTTPS**: All API communications over TLS
- **Input Validation**: Server-side validation for all inputs
- **SQL Injection**: Not applicable (NoSQL database)

### 5.3 API Security
- **CORS**: Configured for frontend domain only
- **Rate Limiting**: AWS API Gateway throttling
- **Input Sanitization**: All user inputs sanitized
- **Error Handling**: Generic error messages (no data leakage)

## 6. Performance & Scalability

### 6.1 Performance Targets
- Page load time: < 3 seconds
- Question transitions: < 1 second
- Score calculation: < 2 seconds
- API response time: < 2 seconds

### 6.2 Scalability Design
- **Auto-scaling**: Lambda functions scale automatically
- **Database**: DynamoDB on-demand billing mode
- **CDN**: CloudFront for static assets and images
- **Caching**: Browser caching for static content

### 6.3 Optimization Strategies
- **Database**: Optimized access patterns and indexes
- **Images**: S3 with CloudFront distribution
- **API**: Minimal payload sizes
- **Frontend**: Code splitting and lazy loading

## 7. Deployment Architecture

### 7.1 Infrastructure Components
```
AWS Account
├── API Gateway (REST APIs)
├── Lambda Functions (Python runtime)
├── DynamoDB Table (single table)
├── S3 Bucket (image storage)
├── CloudFront Distribution (CDN)
├── Route 53 (DNS)
└── CloudWatch (monitoring & logging)
```

### 7.2 Environment Strategy
- **Development**: Separate AWS resources with dev prefix
- **Staging**: Production-like environment for testing
- **Production**: Full-scale deployment with monitoring

### 7.3 Deployment Process
- **Build Scripts**: Automated deployment scripts for Jenkins
- **Infrastructure as Code**: CloudFormation templates
- **Database Migration**: Scripts for schema updates
- **Zero-Downtime**: Blue-green deployment capability

## 8. API Specifications

### 8.1 Authentication APIs
```
POST /auth/register
Body: { email, password }
Response: { message, userId }

POST /auth/login
Body: { email, password }
Response: { token, userId, expiresIn }

GET /auth/validate
Headers: { Authorization: Bearer <token> }
Response: { valid, userId }
```

### 8.2 Quiz APIs
```
GET /topics
Response: { topics: [{ id, name, questionCount }] }

POST /quiz/start
Body: { topicId, userId }
Response: { quizId, questions: [{ id, text, options, image? }] }

POST /quiz/submit
Body: { quizId, answers: [{ questionId, selectedOption }] }
Response: { score, totalQuestions, leaderboardPosition }
```

### 8.3 Score APIs
```
GET /scores/user/{userId}
Response: { scores: [{ score, topic, timestamp }] }

GET /leaderboard
Query: { topic?, limit? }
Response: { rankings: [{ userId, score, rank, timestamp }] }
```

## 9. Database Schema Design

### 9.1 Access Patterns
1. Get user by email (login)
2. Get questions by topic (quiz generation)
3. Save user score (score persistence)
4. Get leaderboard rankings (leaderboard display)
5. CRUD operations on questions (admin)

### 9.2 Entity Definitions
```
User Entity:
PK: USER#{email}
SK: PROFILE
Data: { userId, hashedPassword, createdAt }

Question Entity:
PK: TOPIC#{topicName}
SK: QUESTION#{questionId}
Data: { text, options, correctAnswer, imageUrl? }

Score Entity:
PK: USER#{userId}
SK: SCORE#{timestamp}
Data: { score, topicId, quizId }
GSI1PK: LEADERBOARD#{topicId}
GSI1SK: SCORE#{score}#{timestamp}
```

## 10. Monitoring & Operations

### 10.1 Logging Strategy
- **Application Logs**: CloudWatch Logs for Lambda functions
- **Access Logs**: API Gateway access logging
- **Error Tracking**: Structured error logging with context
- **Performance Logs**: Response time and throughput metrics

### 10.2 Monitoring & Alerts
- **Page Load Time**: Alert if > 4 seconds
- **API Response Time**: Alert if > 2 seconds
- **Error Rate**: Alert if > 5% error rate
- **Database Performance**: Monitor read/write capacity

### 10.3 Backup & Recovery
- **Database**: Daily automated backups
- **Images**: S3 versioning enabled
- **Code**: Git repository with tagged releases
- **Configuration**: Infrastructure as Code templates

## 11. Error Handling & Resilience

### 11.1 Error Handling Strategy
- **Quiz Failures**: Quit quiz and reset session
- **Network Issues**: Display error message, allow retry
- **API Failures**: Generic error messages to user
- **Timer Failures**: Quit quiz (no graceful degradation)

### 11.2 Resilience Patterns
- **Eventual Consistency**: Acceptable for leaderboard updates
- **No Retries**: Fail fast approach as per requirements
- **Circuit Breaker**: Not implemented (Phase 1 simplicity)
- **Graceful Degradation**: Not implemented as per requirements

## 12. Development Guidelines

### 12.1 Code Organization
```
Frontend (React):
src/
├── components/
├── services/
├── hooks/
├── utils/
└── styles/

Backend (Python):
functions/
├── auth/
├── quiz/
├── scores/
├── admin/
└── shared/
```

### 12.2 Development Standards
- **Code Style**: ESLint for React, Black for Python
- **Testing**: Unit tests for business logic
- **Documentation**: API documentation with examples
- **Version Control**: Git with feature branch workflow

---

## 13. Future Considerations (Phase 2)

### 13.1 Planned Enhancements
- AWS Cognito integration for authentication
- Load testing with 1000 concurrent users
- Enhanced monitoring and APM
- Social media login integration
- Advanced leaderboard features

### 13.2 Scalability Improvements
- DynamoDB Global Tables for multi-region
- Enhanced caching strategies
- API rate limiting and throttling
- Advanced error handling and retries

---

*Document Version: 1.0*
*Created: [Current Date]*
*Status: Complete*
*Architecture Pattern: Serverless Microservices*
*Target Scale: 1000 Concurrent Users*
