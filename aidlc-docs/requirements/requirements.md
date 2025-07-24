# Simple Quiz Application - Requirements Document

## 1. Functional Requirements

### 1.1 User Authentication & Management
**FR-001** User Registration
- **Priority:** High
- **Description:** Users must be able to create accounts with email and password
- **Acceptance Criteria:**
  - User can register with valid email and password
  - System validates email format and password strength
  - Duplicate email registration is prevented

**FR-002** User Login
- **Priority:** High  
- **Description:** Registered users must be able to login to access quizzes
- **Acceptance Criteria:**
  - User can login with valid credentials
  - Invalid credentials show appropriate error message
  - User session is maintained during quiz

### 1.2 Topic Selection
**FR-003** Topic Display
- **Priority:** High
- **Description:** System displays available quiz topics for user selection
- **Acceptance Criteria:**
  - Available topics: General Knowledge, Physics, Chemistry, Bollywood Movies, Hollywood Movies
  - Topics are displayed in a clear, selectable format
  - Only admin-managed topics are shown

**FR-004** Topic Selection
- **Priority:** High
- **Description:** User can select one topic to start a quiz
- **Acceptance Criteria:**
  - User can select exactly one topic per quiz session
  - Selection triggers quiz initialization
  - Selected topic determines question pool

### 1.3 Quiz Functionality
**FR-005** Quiz Initialization
- **Priority:** High
- **Description:** System generates a 10-question quiz from selected topic
- **Acceptance Criteria:**
  - Exactly 10 questions are selected from topic question pool
  - Each question has exactly 4 multiple-choice options
  - Questions may include images
  - Questions are presented in random order

**FR-006** Question Display
- **Priority:** High
- **Description:** System displays one question at a time with timer
- **Acceptance Criteria:**
  - Question text and image (if applicable) are clearly displayed
  - 4 answer options are presented as selectable choices
  - 15-second countdown timer is visible and functional
  - Question number indicator shows progress (e.g., "Question 3 of 10")

**FR-007** Answer Selection
- **Priority:** High
- **Description:** User can select one answer per question within time limit
- **Acceptance Criteria:**
  - User can select exactly one answer option
  - Selection is visually indicated
  - No answer review or change is allowed
  - Timer auto-advances to next question when expired (no answer recorded)

**FR-008** Quiz Progression
- **Priority:** High
- **Description:** System automatically progresses through all 10 questions
- **Acceptance Criteria:**
  - After answer selection or timer expiry, next question loads automatically
  - No pause, resume, or navigation between questions
  - Quiz cannot be retaken once completed
  - Progress indicator shows current question number

### 1.4 Scoring & Results
**FR-009** Score Calculation
- **Priority:** High
- **Description:** System calculates final score based on correct answers
- **Acceptance Criteria:**
  - Each correct answer awards exactly 1 point
  - Maximum possible score is 10 points
  - No partial credit or negative scoring
  - Unanswered questions score 0 points

**FR-010** Results Display
- **Priority:** High
- **Description:** System displays final score after quiz completion
- **Acceptance Criteria:**
  - Score is displayed as "X out of 10" format
  - Results are shown immediately after last question
  - No detailed answer explanations provided
  - No pass/fail threshold applied

**FR-011** Score Persistence
- **Priority:** High
- **Description:** System saves user scores for leaderboard and history
- **Acceptance Criteria:**
  - User score is saved with timestamp and topic
  - Score is linked to user account
  - Historical scores are maintained
  - Data persists across sessions

### 1.5 Leaderboard
**FR-012** Leaderboard Display
- **Priority:** Medium
- **Description:** System displays leaderboard after quiz completion
- **Acceptance Criteria:**
  - User is placed on leaderboard based on score
  - Leaderboard shows user rankings
  - User can view their position relative to others
  - Leaderboard updates in real-time

### 1.6 Admin Functions
**FR-013** Question Management
- **Priority:** High
- **Description:** Admin can manage questions and topics
- **Acceptance Criteria:**
  - Admin can add/edit/delete questions
  - Admin can assign questions to topics
  - Admin can upload images for questions
  - Admin can manage the 5 specified topics



---
*Document Version: 1.0*
*Created: [Current Date]*
*Status: Complete*
