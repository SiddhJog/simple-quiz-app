# Component-Story Mapping

## Overview
This document maps each user story to the relevant system components, showing which components are responsible for implementing each story's functionality.

## Story-Component Matrix

### User Authentication & Access Stories

#### US-001: User ID Entry
**Components Involved**:
- **Frontend**: `UserAuthPage` (Primary)
- **Backend**: `user_module.py` (validation), `api_router.py` (endpoint)
- **Database**: Users table (uniqueness check)

**Component Responsibilities**:
- `UserAuthPage`: Input form, client-side validation, error display
- `user_module.py`: Server-side ID validation and format checking
- `api_router.py`: Handle authentication API endpoint
- Users table: Store and check user ID uniqueness

#### US-002: User ID Validation
**Components Involved**:
- **Frontend**: `UserAuthPage` (Primary)
- **Backend**: `user_module.py` (Primary), `validation_module.py`, `api_router.py`
- **Database**: Users table

**Component Responsibilities**:
- `UserAuthPage`: Display validation errors, retry interface
- `user_module.py`: Uniqueness validation logic
- `validation_module.py`: Input sanitization and format validation
- `api_router.py`: Validation endpoint and error responses
- Users table: Uniqueness constraint enforcement

#### US-003: Session-Based Access
**Components Involved**:
- **Frontend**: `UserAuthPage` (Primary), All other pages (session checking)
- **Backend**: `user_module.py` (Primary), `api_router.py`

**Component Responsibilities**:
- `UserAuthPage`: Session cookie creation after successful validation
- All frontend pages: Session cookie validation on page load
- `user_module.py`: Session management logic
- `api_router.py`: Session validation middleware

### Dashboard & Performance Tracking Stories

#### US-004: Dashboard Overview
**Components Involved**:
- **Frontend**: `DashboardPage` (Primary)
- **Backend**: `user_module.py` (Primary), `api_router.py`
- **Database**: User Statistics table, Quiz Results table

**Component Responsibilities**:
- `DashboardPage`: Layout, responsive design, data display
- `user_module.py`: Aggregate user statistics
- `api_router.py`: Dashboard data API endpoint
- Database tables: Store and retrieve user performance data

#### US-005: Recent Quiz Scores
**Components Involved**:
- **Frontend**: `DashboardPage` (Primary)
- **Backend**: `user_module.py` (Primary), `database_module.py`
- **Database**: Quiz Results table (Primary)

**Component Responsibilities**:
- `DashboardPage`: Display last 5 scores with formatting
- `user_module.py`: Retrieve and format recent scores
- `database_module.py`: Query last 5 quiz results
- Quiz Results table: Store chronological quiz data

#### US-006: Average Score Display
**Components Involved**:
- **Frontend**: `DashboardPage` (Primary)
- **Backend**: `user_module.py` (Primary), `database_module.py`
- **Database**: User Statistics table (Primary)

**Component Responsibilities**:
- `DashboardPage`: Display average score with proper formatting
- `user_module.py`: Calculate and retrieve average scores
- `database_module.py`: Query and update statistics
- User Statistics table: Store calculated averages

#### US-007: Topic Performance Statistics
**Components Involved**:
- **Frontend**: `DashboardPage` (Primary)
- **Backend**: `user_module.py` (Primary), `database_module.py`
- **Database**: Quiz Results table (Primary), Topics table

**Component Responsibilities**:
- `DashboardPage`: Visual representation of topic performance
- `user_module.py`: Calculate topic-wise statistics
- `database_module.py`: Query results by topic
- Quiz Results table: Store topic-linked results

#### US-008: Average Response Time
**Components Involved**:
- **Frontend**: `DashboardPage` (Primary)
- **Backend**: `user_module.py` (Primary), `database_module.py`
- **Database**: Quiz Results table (Primary), User Statistics table

**Component Responsibilities**:
- `DashboardPage`: Display response time metrics
- `user_module.py`: Calculate average response times
- `database_module.py`: Query timing data
- Database tables: Store and aggregate timing information

### Quiz Selection & Initiation Stories

#### US-009: Start New Quiz Button
**Components Involved**:
- **Frontend**: `DashboardPage` (Primary)
- **Backend**: None (navigation only)

**Component Responsibilities**:
- `DashboardPage`: Button display, navigation to topic selection

#### US-010: Topic Selection
**Components Involved**:
- **Frontend**: `TopicSelectionPage` (Primary)
- **Backend**: `quiz_module.py` (Primary), `api_router.py`
- **Database**: Topics table (Primary)

**Component Responsibilities**:
- `TopicSelectionPage`: Display topics, selection interface
- `quiz_module.py`: Retrieve available topics
- `api_router.py`: Topics API endpoint
- Topics table: Store topic information

#### US-011: Random Question Selection
**Components Involved**:
- **Frontend**: `QuizPage` (receives questions)
- **Backend**: `quiz_module.py` (Primary), `database_module.py`
- **Database**: Questions table (Primary)

**Component Responsibilities**:
- `quiz_module.py`: Random selection algorithm, duplicate prevention
- `database_module.py`: Query questions by topic
- Questions table: Store question bank
- `QuizPage`: Receive and display selected questions

### Quiz Taking Experience Stories

#### US-012: Question Display
**Components Involved**:
- **Frontend**: `QuizPage` (Primary), `QuestionDisplay` component
- **Backend**: `quiz_module.py`, `api_router.py`
- **Database**: Questions table

**Component Responsibilities**:
- `QuizPage`: Overall quiz flow management
- `QuestionDisplay`: Render individual questions
- `quiz_module.py`: Provide question data
- `api_router.py`: Question API endpoint

#### US-013: Answer Selection
**Components Involved**:
- **Frontend**: `QuizPage` (Primary), `AnswerChoices` component
- **Backend**: None (client-side only)

**Component Responsibilities**:
- `QuizPage`: Manage answer state
- `AnswerChoices`: Handle selection interface and feedback

#### US-014: Question Timer
**Components Involved**:
- **Frontend**: `QuizPage` (Primary), `Timer` component
- **Backend**: None (client-side timer)

**Component Responsibilities**:
- `QuizPage`: Timer integration with quiz flow
- `Timer`: 20-second countdown implementation

#### US-015: Timer Expiration Handling
**Components Involved**:
- **Frontend**: `QuizPage` (Primary), `Timer` component
- **Backend**: None (client-side handling)

**Component Responsibilities**:
- `QuizPage`: Handle timer expiration, mark as incorrect, progress to next
- `Timer`: Trigger expiration events

#### US-016: Forward-Only Navigation
**Components Involved**:
- **Frontend**: `QuizPage` (Primary)
- **Backend**: None (client-side navigation)

**Component Responsibilities**:
- `QuizPage`: Implement forward-only logic, disable back navigation

#### US-017: Quiz Progress Indicator
**Components Involved**:
- **Frontend**: `QuizPage` (Primary), `ProgressBar` component
- **Backend**: None (client-side display)

**Component Responsibilities**:
- `QuizPage`: Track current question number
- `ProgressBar`: Visual progress representation

### Scoring & Results Stories

#### US-018: Score Calculation
**Components Involved**:
- **Frontend**: `QuizPage` (client-side tracking)
- **Backend**: `quiz_module.py` (Primary), `api_router.py`
- **Database**: None (calculation only)

**Component Responsibilities**:
- `QuizPage`: Track user answers
- `quiz_module.py`: Validate answers and calculate final score
- `api_router.py`: Score calculation API endpoint

#### US-019: Final Score Display
**Components Involved**:
- **Frontend**: `ResultsPage` (Primary)
- **Backend**: `quiz_module.py`, `api_router.py`
- **Database**: Quiz Results table

**Component Responsibilities**:
- `ResultsPage`: Display final score with formatting
- `quiz_module.py`: Provide score data
- Quiz Results table: Store final results

#### US-020: Correct Answer Review
**Components Involved**:
- **Frontend**: `ResultsPage` (Primary)
- **Backend**: `quiz_module.py` (Primary), `api_router.py`
- **Database**: Questions table

**Component Responsibilities**:
- `ResultsPage`: Display questions with correct answers
- `quiz_module.py`: Provide question and answer data
- Questions table: Store correct answer information

#### US-021: Results Storage
**Components Involved**:
- **Frontend**: `QuizPage` (trigger storage)
- **Backend**: `quiz_module.py` (Primary), `database_module.py`
- **Database**: Quiz Results table (Primary), User Statistics table

**Component Responsibilities**:
- `quiz_module.py`: Process and store quiz results
- `database_module.py`: Execute database operations
- Quiz Results table: Store individual quiz data
- User Statistics table: Update aggregated statistics

### Error Handling Stories

#### US-022: Duplicate User ID Error
**Components Involved**:
- **Frontend**: `UserAuthPage` (Primary), `ErrorBoundary`
- **Backend**: `user_module.py`, `api_router.py`
- **Database**: Users table

**Component Responsibilities**:
- `UserAuthPage`: Display error message, provide retry interface
- `user_module.py`: Detect duplicate ID condition
- `api_router.py`: Return appropriate error response

#### US-023: Network Connection Error
**Components Involved**:
- **Frontend**: `NetworkError` component (Primary), All pages
- **Backend**: None (network-level issue)

**Component Responsibilities**:
- `NetworkError`: Display connection error message
- All frontend pages: Detect and handle network failures

#### US-024: Quiz Session Loss
**Components Involved**:
- **Frontend**: `QuizPage` (Primary), `ErrorBoundary`
- **Backend**: None (client-side session management)

**Component Responsibilities**:
- `QuizPage`: Detect session loss, display appropriate message
- `ErrorBoundary`: Handle unexpected session termination

### Administrative Functions

#### US-025: Question Bank Setup
**Components Involved**:
- **Backend**: `database_module.py` (Primary)
- **Database**: Questions table (Primary), Topics table
- **Script**: Database population script

**Component Responsibilities**:
- Database script: Populate question banks with 100+ questions per topic
- `database_module.py`: Provide database access functions
- Questions table: Store question data
- Topics table: Store topic definitions

## Component Priority Matrix

### High Priority Components (Core User Journey)
1. `UserAuthPage` - Entry point for all users
2. `DashboardPage` - Central hub for user interaction
3. `QuizPage` - Core quiz-taking functionality
4. `user_module.py` - User management and validation
5. `quiz_module.py` - Quiz logic and scoring

### Medium Priority Components (Enhanced Experience)
1. `TopicSelectionPage` - Topic selection interface
2. `ResultsPage` - Results display and review
3. `database_module.py` - Data persistence
4. `api_router.py` - API endpoint management

### Lower Priority Components (Support Functions)
1. `validation_module.py` - Input validation
2. Error handling components
3. Sub-components (Timer, ProgressBar, etc.)

## Cross-Component Stories
Stories that require coordination between multiple components:

- **US-011**: Random Question Selection (Quiz Module + Database + Frontend)
- **US-021**: Results Storage (Frontend + Quiz Module + Database + Statistics)
- **US-004**: Dashboard Overview (Multiple backend modules + Database + Frontend)

## Component Coverage Validation
✅ All 25 user stories are mapped to appropriate components
✅ Each component has clear responsibilities
✅ Dependencies between components are identified
✅ No orphaned components (all serve user stories)
✅ No uncovered user stories (all have implementing components)
