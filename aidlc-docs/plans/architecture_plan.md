# Software Architecture Design Plan

## Overview
This plan outlines the steps to analyze requirements, user stories, and personas to create a comprehensive software architecture design for the quiz application.

## Plan Steps

### Phase 1: Requirements Analysis & Understanding
- [x] **1.1 Document Analysis**
  - [x] Review requirements.md for functional and non-functional requirements
  - [x] Analyze user stories in stories.md for component identification
  - [x] Study personas.md to understand user interaction patterns
  - [x] Extract technical constraints and specifications

- [x] **1.2 System Boundary Definition**
  - [x] Identify system actors (users, administrators)
  - [x] Define system scope and boundaries
  - [x] List external dependencies and integrations
  - [x] Identify data flow patterns

### Phase 2: Component Identification & Design
- [x] **2.1 Component Identification**
  - [x] Identify frontend components based on user interactions
  - [x] Identify backend components based on business logic
  - [x] Identify data storage components
  - [x] Define component responsibilities and boundaries

- [x] **2.2 Component Dependency Mapping**
  - [x] Map dependencies between components
  - [x] Identify data flow between components
  - [x] Define component interfaces and contracts
  - [x] Create visual dependency diagrams

### Phase 3: Story-Component Mapping
- [x] **3.1 Story Analysis**
  - [x] Group user stories by functional areas
  - [x] Map each story to relevant components
  - [x] Identify cross-component stories
  - [x] Validate component coverage

- [x] **3.2 Component-Story Matrix**
  - [x] Create mapping matrix of stories to components
  - [x] Identify component priorities based on story importance
  - [x] Validate all stories are covered by components
  - [x] Document component-story relationships

### Phase 4: Domain Model Creation
- [x] **4.1 Domain Entity Identification**
  - [x] Identify core business entities from requirements
  - [x] Define entity attributes and relationships
  - [x] Model entity lifecycle and state transitions
  - [x] Validate entities against user stories

- [x] **4.2 Domain Model Documentation**
  - [x] Create entity relationship diagrams
  - [x] Document entity attributes and constraints
  - [x] Define business rules and validations
  - [x] Model data persistence requirements

### Phase 5: API Specification Design
- [x] **5.1 API Endpoint Identification**
  - [x] Identify required API endpoints from user stories
  - [x] Group endpoints by functional areas
  - [x] Define HTTP methods and URL patterns
  - [x] Specify request/response data structures

- [x] **5.2 API Documentation**
  - [x] Document endpoint specifications
  - [x] Define request/response schemas
  - [x] Specify error handling and status codes
  - [x] Include authentication and validation requirements

### Phase 6: Architecture Documentation & Review
- [x] **6.1 Document Creation**
  - [x] Create components.md with component definitions and dependencies
  - [x] Create component-story-map.md with story mappings
  - [x] Create domain-model.md with entity models
  - [x] Create api-specification.md with API documentation

- [x] **6.2 Architecture Validation**
  - [x] Validate architecture against requirements
  - [x] Check component coverage of all user stories
  - [x] Verify API completeness for all user interactions
  - [x] Review for scalability and performance considerations

## Clarification Questions

### System Architecture Decisions
1. **Technology Stack**: What technology stack should be used for frontend and backend? (e.g., React/Vue.js for frontend, Node.js/Python/Java for backend)
[Answer]: Python backend, ReactJS Frontend (although we may choose to change front end tech later)

2. **Database Choice**: What type of database should be used? (e.g., PostgreSQL, MySQL, MongoDB) and should it be relational or NoSQL?
[Answer]: mysql

3. **Deployment Architecture**: Should this be designed as a monolithic application or microservices? What about containerization (Docker)?
[Answer]: monolithic, frontand and backend separate


4. **Authentication Approach**: Since users enter unique IDs without registration, how should session management be handled? Should we use JWT tokens, session cookies, or in-memory sessions?
[Answer]: session cookies

**Explanation**: For your use case, here are the options ranked by ease of implementation:
1. **In-memory sessions (Easiest)**: Store user sessions in server memory. Simple to implement but doesn't scale across multiple server instances.
2. **Session cookies**: Store session data in browser cookies. Easy to implement and works well for single-server deployments.
3. **JWT tokens**: Stateless tokens that contain user info. More complex but better for scaling.

**Recommendation**: Use in-memory sessions for Phase 1 since you have a monolithic architecture and it's the simplest approach.

### Component Granularity
5. **Frontend Component Structure**: Should frontend components be organized by pages/views or by functional areas? How granular should the component breakdown be?
[Answer]: pages

6. **Backend Service Organization**: Should backend logic be organized into separate services (user service, quiz service, etc.) or kept as a single service with multiple modules?
[Answer]: single backend application individual modules in it

7. **API Design Pattern**: Should we follow REST principles strictly, or consider GraphQL for more flexible data fetching?
[Answer]: REST

### Data Management
8. **Session Storage**: Since quiz progress isn't saved, should we use in-memory storage for active quiz sessions, or should temporary data be stored in the database?
[Answer]: in-memory

9. **Caching Strategy**: Should we implement caching for frequently accessed data like questions and user statistics? If so, what caching mechanism?
[Answer]: not for phase 1

10. **Data Consistency**: How should we handle concurrent access to user data and ensure data consistency for scoring and statistics?
[Answer]: WE are ok with eventual consistency. Assume users access from only 1 device for now.

**Explanation**: Concurrent access scenarios in your quiz app:
1. **Same user, multiple devices**: User opens app on phone and laptop simultaneously with same user ID
2. **Database writes**: Multiple users finishing quizzes at the same time, updating statistics
3. **Question bank access**: Multiple users requesting random questions from the same topic simultaneously

**Impact**: Could cause data corruption in user statistics or duplicate question selection.

**Recommendation**: Since users enter unique IDs per session and quiz progress isn't saved, concurrent access is minimal. Simple database transactions will handle the few write operations (saving quiz results).

### Performance & Scalability
11. **Real-time Requirements**: The 20-second timer needs to be accurate - should this be handled client-side only, server-side only, or hybrid approach?
[Answer]: client side only


12. **Question Randomization**: Should question randomization happen at the database level or application level? How do we ensure good performance with large question banks?
[Answer]: application level

13. **Concurrent User Handling**: With up to 1000 concurrent users, what strategies should be implemented for load balancing and resource management?
[Answer]: horizontal scaling

### Security Considerations
14. **Input Validation**: What level of input validation and sanitization should be implemented for user IDs and quiz responses?
[Answer]: cross site scripting validation, sql injection for textfields, quiz response is not text its a selection of one of the 4 options

15. **API Security**: Should we implement rate limiting, CORS policies, and other security measures for the API endpoints?
[Answer]: not for phase 1

16. **Data Protection**: How should we protect quiz questions and answers from being exposed to the client before they're needed?
[Answer]: ignore this issue for now

**Explanation**: Security concerns for quiz data:
1. **Question exposure**: If all questions are sent to client at once, users could inspect browser dev tools to see upcoming questions
2. **Answer exposure**: Sending correct answers to client allows users to cheat by viewing network requests
3. **Question bank exposure**: Users could potentially access entire question database

**Protection strategies**:
1. **One question at a time**: API sends only current question without correct answer
2. **Answer validation on server**: Client sends selected answer, server validates and returns result
3. **No answer data to client**: Correct answers never sent to frontend during quiz

**Recommendation**: Send questions one at a time without correct answers, validate answers server-side.

## Confirmed Architecture Decisions
- **Technology Stack**: Python backend, ReactJS frontend
- **Database**: MySQL
- **Architecture**: Monolithic (separate frontend and backend)
- **API Design**: REST
- **Session Management**: Session cookies
- **Timer Implementation**: Client-side only
- **Question Randomization**: Application level
- **Session Storage**: In-memory for active quiz sessions
- **Caching**: Not implemented in Phase 1
- **Data Consistency**: Eventual consistency, single device per user
- **Security**: XSS and SQL injection validation, no rate limiting in Phase 1
- **Data Protection**: Not implemented in Phase 1
- **Scaling**: Horizontal scaling approach
- **Component Organization**: Frontend by pages, backend by modules
- **Input Validation**: XSS and SQL injection protection for text fields frontend and backend)
- **API Design**: REST
- **Session Management**: In-memory sessions (recommended)
- **Timer Implementation**: Client-side only
- **Question Randomization**: Application level
- **Session Storage**: In-memory for active quiz sessions
- **Caching**: Not implemented in Phase 1
- **Security**: XSS and SQL injection validation, no rate limiting in Phase 1
- **Scaling**: Horizontal scaling approach
- **Component Organization**: Frontend by pages, backend by modules
- **Input Validation**: XSS and SQL injection protection for text fields

## Deliverables
1. **components.md** - Component definitions, responsibilities, and dependency maps
2. **component-story-map.md** - Mapping of user stories to components
3. **domain-model.md** - Domain entities, relationships, and business rules
4. **api-specification.md** - Complete API endpoint documentation

## Success Criteria
- All user stories mapped to appropriate components
- Complete component dependency mapping with clear interfaces
- Domain model covers all business entities and relationships
- API specification supports all user interactions
- Architecture supports scalability and performance requirements
- Design is maintainable and follows best practices

## Next Steps
1. **Review this plan** and provide feedback on approach and scope
2. **Answer clarification questions** to guide architecture decisions
3. **Approve the plan** to proceed with implementation
4. **Proceed with architecture design** once plan is approved
