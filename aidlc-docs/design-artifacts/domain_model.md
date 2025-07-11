# Domain Model

## Overview
This document defines the domain model for the Simple Quiz Application, including core business entities, their relationships, domain services, and business rules.

## Core Domain Entities

### 1. User Entity
**Description**: Represents a user of the quiz application (both regular users and admins)

**Attributes**:
- `userId`: String (Primary identifier)
- `email`: String (Unique, used for login)
- `passwordHash`: String (Encrypted password)
- `role`: Enum (USER, ADMIN)
- `createdAt`: DateTime
- `lastLoginAt`: DateTime
- `isActive`: Boolean

**Business Rules**:
- Email must be unique across all users
- Password must meet policy: 8+ chars, 1 uppercase, 1 lowercase, 1 special character
- Users can have multiple quiz sessions but only one active session at a time

### 2. Topic Entity
**Description**: Represents quiz topics/categories

**Attributes**:
- `topicId`: String (Primary identifier)
- `name`: String (Topic name)
- `description`: String
- `isActive`: Boolean
- `questionCount`: Integer (Derived)
- `createdAt`: DateTime
- `updatedAt`: DateTime

**Business Rules**:
- Topic names are fixed: General Knowledge, Physics, Chemistry, Bollywood Movies, Hollywood Movies
- Topics cannot be deleted, only deactivated
- Must have at least 10 questions to be available for quizzes

### 3. Question Entity
**Description**: Represents individual quiz questions

**Attributes**:
- `questionId`: String (Primary identifier)
- `topicId`: String (Foreign key to Topic)
- `questionText`: String
- `options`: Array[String] (Exactly 4 options)
- `correctAnswerIndex`: Integer (0-3)
- `imageUrl`: String (Optional, S3 URL)
- `difficulty`: Enum (EASY, MEDIUM, HARD) - Future use
- `isActive`: Boolean
- `createdAt`: DateTime
- `updatedAt`: DateTime
- `createdBy`: String (Admin userId)

**Business Rules**:
- Must have exactly 4 answer options
- Correct answer index must be between 0-3
- Question text is required and non-empty
- Images are optional, max 1MB, JPG/PNG only
- Questions can be soft-deleted (isActive = false)

### 4. Quiz Entity
**Description**: Represents a quiz session instance

**Attributes**:
- `quizId`: String (Primary identifier)
- `userId`: String (Foreign key to User)
- `topicId`: String (Foreign key to Topic)
- `questions`: Array[QuizQuestion] (10 questions)
- `status`: Enum (IN_PROGRESS, COMPLETED, ABANDONED)
- `startedAt`: DateTime
- `completedAt`: DateTime
- `timeLimit`: Integer (15 seconds per question)
- `currentQuestionIndex`: Integer

**Business Rules**:
- Quiz must have exactly 10 questions
- Questions are randomly selected from topic
- Quiz expires if not completed within reasonable time
- Cannot restart or retake completed quizzes
- User can have only one active quiz at a time

### 5. QuizQuestion Entity
**Description**: Represents a question within a specific quiz instance

**Attributes**:
- `questionId`: String (Reference to Question)
- `questionText`: String (Snapshot)
- `options`: Array[String] (Snapshot of 4 options)
- `imageUrl`: String (Optional)
- `userAnswer`: Integer (0-3, null if not answered)
- `answeredAt`: DateTime
- `timeSpent`: Integer (seconds)

**Business Rules**:
- Stores snapshot of question data at quiz time
- User can select only one answer (0-3)
- Answer cannot be changed once selected
- Auto-advances after 15 seconds if no answer

### 6. Score Entity
**Description**: Represents quiz results and scoring

**Attributes**:
- `scoreId`: String (Primary identifier)
- `userId`: String (Foreign key to User)
- `quizId`: String (Foreign key to Quiz)
- `topicId`: String (Foreign key to Topic)
- `score`: Integer (0-10)
- `totalQuestions`: Integer (Always 10)
- `correctAnswers`: Integer
- `completedAt`: DateTime
- `timeTaken`: Integer (Total seconds)

**Business Rules**:
- Score is calculated as number of correct answers
- Maximum score is 10 (one point per correct answer)
- No partial credit or negative scoring
- Unanswered questions count as incorrect
- Scores are immutable once saved

### 7. LeaderboardEntry Entity
**Description**: Represents leaderboard rankings

**Attributes**:
- `entryId`: String (Primary identifier)
- `userId`: String (Foreign key to User)
- `topicId`: String (Foreign key to Topic)
- `score`: Integer
- `rank`: Integer (Calculated)
- `userName`: String (Display name)
- `achievedAt`: DateTime

**Business Rules**:
- Rankings are calculated based on highest scores
- Ties are broken by earliest completion time
- Leaderboard is eventually consistent
- Shows top performers and user's current position

## Entity Relationships

### Relationship Diagram
```
User (1) -----> (0..*) Quiz
User (1) -----> (0..*) Score
User (1) -----> (0..*) LeaderboardEntry

Topic (1) -----> (0..*) Question
Topic (1) -----> (0..*) Quiz
Topic (1) -----> (0..*) Score
Topic (1) -----> (0..*) LeaderboardEntry

Quiz (1) -----> (10) QuizQuestion
Quiz (1) -----> (1) Score

Question (1) -----> (0..*) QuizQuestion
```

### Detailed Relationships

**User → Quiz** (One-to-Many)
- A user can take multiple quizzes over time
- Each quiz belongs to exactly one user
- Cascade: User deletion should handle quiz cleanup

**User → Score** (One-to-Many)
- A user can have multiple scores (quiz history)
- Each score belongs to exactly one user
- Cascade: Preserve scores for analytics even if user deleted

**Topic → Question** (One-to-Many)
- A topic contains multiple questions
- Each question belongs to exactly one topic
- Cascade: Topic deactivation affects question availability

**Quiz → QuizQuestion** (One-to-Many, Fixed 10)
- Each quiz contains exactly 10 quiz questions
- Quiz questions are snapshots of original questions
- Cascade: Quiz deletion removes quiz questions

**Quiz → Score** (One-to-One)
- Each completed quiz has exactly one score
- Score is created when quiz is completed
- Cascade: Quiz deletion removes associated score

## Domain Services

### 1. AuthenticationService
**Responsibilities**:
- User registration and validation
- Password hashing and verification
- JWT token generation and validation
- Session management

**Key Operations**:
```
registerUser(email, password) -> User
authenticateUser(email, password) -> AuthToken
validateToken(token) -> User
hashPassword(password) -> String
```

**Domain Events**:
- UserRegistered
- UserAuthenticated
- UserLoggedOut

### 2. QuizService
**Responsibilities**:
- Quiz initialization and management
- Question selection and randomization
- Quiz progression and timing
- Answer submission and validation

**Key Operations**:
```
initializeQuiz(userId, topicId) -> Quiz
getNextQuestion(quizId) -> QuizQuestion
submitAnswer(quizId, questionIndex, answer) -> Boolean
completeQuiz(quizId) -> Score
```

**Domain Events**:
- QuizStarted
- QuestionAnswered
- QuizCompleted
- QuizAbandoned

### 3. ScoringService
**Responsibilities**:
- Score calculation and validation
- Score persistence and history
- Leaderboard updates

**Key Operations**:
```
calculateScore(quiz) -> Score
saveScore(score) -> Boolean
getUserScoreHistory(userId) -> List[Score]
updateLeaderboard(score) -> Boolean
```

**Domain Events**:
- ScoreCalculated
- ScoreSaved
- LeaderboardUpdated

### 4. QuestionManagementService
**Responsibilities**:
- Question CRUD operations
- Image upload and management
- Topic management
- Content validation

**Key Operations**:
```
createQuestion(questionData) -> Question
updateQuestion(questionId, questionData) -> Question
deleteQuestion(questionId) -> Boolean
uploadQuestionImage(questionId, imageData) -> String
getQuestionsByTopic(topicId) -> List[Question]
```

**Domain Events**:
- QuestionCreated
- QuestionUpdated
- QuestionDeleted
- QuestionImageUploaded

### 5. LeaderboardService
**Responsibilities**:
- Leaderboard calculation and ranking
- Real-time leaderboard updates
- User ranking retrieval

**Key Operations**:
```
calculateRankings(topicId) -> List[LeaderboardEntry]
getUserRanking(userId, topicId) -> LeaderboardEntry
getTopPerformers(topicId, limit) -> List[LeaderboardEntry]
updateUserRanking(userId, score) -> LeaderboardEntry
```

**Domain Events**:
- LeaderboardCalculated
- UserRankingUpdated

## Business Rules & Constraints

### Quiz Rules
1. **Question Selection**: Exactly 10 questions randomly selected from chosen topic
2. **Time Limit**: 15 seconds per question, auto-advance on timeout
3. **Answer Policy**: One answer per question, no changes allowed
4. **Completion**: Quiz must be completed in sequence, no skipping
5. **Retake Policy**: No retakes allowed for completed quizzes

### Scoring Rules
1. **Point System**: 1 point per correct answer, maximum 10 points
2. **No Penalties**: No negative scoring for incorrect answers
3. **Unanswered**: Unanswered questions score 0 points
4. **Immutability**: Scores cannot be modified once saved

### User Rules
1. **Registration**: Unique email required, password policy enforced
2. **Session Limit**: One active quiz session per user
3. **Role-Based Access**: Admin users can manage questions and topics

### Content Rules
1. **Question Format**: Exactly 4 multiple-choice options required
2. **Image Policy**: Optional images, 1MB max, JPG/PNG only
3. **Topic Constraint**: Fixed set of 5 topics, cannot be modified
4. **Minimum Questions**: Topics need 10+ questions to be available

### Leaderboard Rules
1. **Ranking Logic**: Highest score wins, ties broken by completion time
2. **Consistency**: Eventually consistent updates acceptable
3. **Visibility**: Users can see their rank and top performers

## Domain Events

### Event Types
```
// Authentication Events
UserRegistered { userId, email, timestamp }
UserAuthenticated { userId, timestamp }
UserLoggedOut { userId, timestamp }

// Quiz Events  
QuizStarted { quizId, userId, topicId, timestamp }
QuestionAnswered { quizId, questionIndex, answer, timestamp }
QuizCompleted { quizId, score, timestamp }
QuizAbandoned { quizId, reason, timestamp }

// Scoring Events
ScoreCalculated { scoreId, userId, score, timestamp }
ScoreSaved { scoreId, userId, topicId, score, timestamp }
LeaderboardUpdated { topicId, userId, newRank, timestamp }

// Content Management Events
QuestionCreated { questionId, topicId, createdBy, timestamp }
QuestionUpdated { questionId, changes, updatedBy, timestamp }
QuestionDeleted { questionId, deletedBy, timestamp }
QuestionImageUploaded { questionId, imageUrl, timestamp }
```

### Event Sourcing Considerations
- Events provide audit trail for all domain changes
- Events enable eventual consistency for leaderboard updates
- Events support future analytics and reporting requirements
- Events allow for system state reconstruction if needed

## Data Consistency Patterns

### Strong Consistency Required
- User authentication and registration
- Quiz question selection and initialization
- Score calculation and saving

### Eventual Consistency Acceptable
- Leaderboard rankings and updates
- Question count per topic
- User activity statistics

### Conflict Resolution
- Quiz sessions: Last writer wins for user preferences
- Leaderboard: Recalculate rankings on conflicts
- Questions: Admin changes override with audit trail

---

*Document Version: 1.0*
*Created: [Current Date]*
*Status: Complete*
*Domain Complexity: Simple with Complete Coverage*
