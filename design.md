# AI Gym Coach – System Design Document

## 1. Architecture Overview

AI Gym Coach follows a three-tier architecture with a React frontend, FastAPI backend, and SQLite database. The system integrates with Kiro API as an external AI service for intelligent workout generation and coaching. The architecture emphasizes modularity, clear separation of concerns, and a closed adaptive feedback loop.

## 2. High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND LAYER                        │
│                      (React Application)                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  User    │  │ Workout  │  │Performance│  │ Profile  │   │
│  │  Auth    │  │ Display  │  │  Logging  │  │ Manager  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │ HTTP/REST
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                        BACKEND LAYER                         │
│                     (FastAPI Application)                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   Auth   │  │ Workout  │  │Adaptation│  │  Injury  │   │
│  │ Service  │  │Generator │  │  Engine  │  │ Handler  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │            AI Integration Service                     │  │
│  │         (Kiro API Client & Prompt Manager)           │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │ SQL
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                       DATABASE LAYER                         │
│                      (SQLite Database)                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Users   │  │ Workouts │  │  Logs    │  │Exercises │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘

                            │ HTTPS
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      EXTERNAL SERVICE                        │
│                         (Kiro API)                           │
└─────────────────────────────────────────────────────────────┘
```

### Frontend Responsibilities
- User authentication and session management
- Workout display and visualization
- Performance logging interface
- Profile management forms
- API communication and state management
- Client-side validation

### Backend Responsibilities
- RESTful API endpoints
- Business logic orchestration
- AI prompt construction and response handling
- Adaptive progression algorithms
- Injury-aware exercise substitution
- Data validation and sanitization
- Authentication and authorization

### Database Responsibilities
- User data persistence
- Workout history storage
- Performance log storage
- Exercise library management
- Relationship management between entities

### AI Engine Responsibilities
- Workout plan generation
- Exercise substitution recommendations
- Coaching explanations
- Progression analysis insights

## 3. Component Design

### 3.1 Frontend Components

#### AuthComponent
- Handles user login and registration
- Manages JWT token storage
- Provides authentication context to child components

#### ProfileManager
- Displays and edits user profile
- Manages injury list
- Updates fitness goals and equipment

#### WorkoutDisplay
- Renders current workout plan
- Shows exercise details (sets, reps, rest)
- Displays AI coaching notes
- Provides workout completion interface

#### PerformanceLogger
- Captures actual sets/reps completed
- Records weight used
- Collects perceived difficulty rating
- Submits performance data to backend

#### TrendAnalyzer
- Visualizes performance trends over time
- Shows progression metrics
- Displays consistency statistics

### 3.2 Backend Services

#### AuthService
- User registration and login
- Password hashing (bcrypt)
- JWT token generation and validation
- User session management

#### WorkoutGenerator
- Constructs AI prompts with user context
- Calls Kiro API for workout generation
- Validates JSON response structure
- Stores generated workouts in database

#### AdaptationEngine
- Analyzes performance logs
- Calculates progression metrics
- Determines difficulty adjustments
- Triggers workout regeneration when needed

#### InjuryHandler
- Identifies conflicting exercises
- Queries exercise database for alternatives
- Uses AI for intelligent substitutions
- Validates substitution safety

#### AIIntegrationService
- Manages Kiro API communication
- Constructs structured prompts
- Handles API errors and retries
- Validates and parses AI responses

## 4. Data Flow Description

### Workout Generation Flow
```
User Request → Backend API → Adaptation Engine (analyze history)
                           ↓
                    Injury Handler (check constraints)
                           ↓
                    AI Integration Service (build prompt)
                           ↓
                    Kiro API (generate workout)
                           ↓
                    Workout Generator (validate & store)
                           ↓
                    Frontend (display workout)
```

### Performance Logging Flow
```
User Logs Workout → Backend API → Validate Input
                                 ↓
                          Store in Database
                                 ↓
                          Adaptation Engine (analyze)
                                 ↓
                    Determine if adjustment needed
                                 ↓
                    Trigger regeneration (if needed)
```

### Adaptive Feedback Loop
```
┌─────────────────────────────────────────────────────┐
│                                                      │
│  Generate Workout → User Performs → Log Performance │
│         ▲                                    │       │
│         │                                    ▼       │
│         └──────── Analyze & Adapt ◄─────────┘       │
│                                                      │
└─────────────────────────────────────────────────────┘
```

## 5. Adaptive Feedback Loop Logic

### Progression Triggers

#### Increase Difficulty When:
- Completion rate ≥ 95% for last 3 workouts
- User rates difficulty ≤ 4/10 for last 2 workouts
- Volume (sets × reps × weight) increases consistently

#### Decrease Difficulty When:
- Completion rate ≤ 70% for last 3 workouts
- User rates difficulty ≥ 9/10 for last 2 workouts
- User skips 2+ consecutive workouts

#### Maintain Current Level When:
- Completion rate between 70-95%
- Difficulty ratings between 5-8/10
- Consistent performance without clear trends

### Adaptation Parameters

**Progressive Overload Methods:**
- Increase weight by 2.5-5%
- Add 1-2 reps per set
- Add 1 additional set
- Decrease rest periods by 10-15 seconds
- Increase exercise complexity

**Regression Methods:**
- Decrease weight by 5-10%
- Reduce reps by 2-3 per set
- Remove 1 set
- Increase rest periods by 15-30 seconds
- Simplify exercise variations

## 6. API Design Overview

### Authentication Endpoints

**POST /api/auth/register**
```json
Request:
{
  "email": "user@example.com",
  "password": "securepass123",
  "name": "John Doe"
}

Response:
{
  "user_id": "uuid",
  "token": "jwt_token"
}
```

**POST /api/auth/login**
```json
Request:
{
  "email": "user@example.com",
  "password": "securepass123"
}

Response:
{
  "user_id": "uuid",
  "token": "jwt_token"
}
```

### User Profile Endpoints

**GET /api/users/{user_id}/profile**
```json
Response:
{
  "user_id": "uuid",
  "name": "John Doe",
  "age": 30,
  "fitness_level": "intermediate",
  "goals": ["strength", "muscle_gain"],
  "injuries": ["lower_back"],
  "equipment": ["dumbbells", "barbell", "bench"]
}
```

**PUT /api/users/{user_id}/profile**
```json
Request:
{
  "fitness_level": "advanced",
  "goals": ["strength"],
  "injuries": [],
  "equipment": ["full_gym"]
}

Response:
{
  "success": true,
  "updated_profile": { ... }
}
```

### Workout Endpoints

**POST /api/workouts/generate**
```json
Request:
{
  "user_id": "uuid"
}

Response:
{
  "workout_id": "uuid",
  "generated_at": "2026-02-15T10:00:00Z",
  "exercises": [
    {
      "name": "Barbell Squat",
      "sets": 4,
      "reps": 8,
      "weight": "135 lbs",
      "rest_seconds": 120,
      "notes": "Focus on depth and form"
    }
  ],
  "coaching_notes": "This workout focuses on compound movements..."
}
```

**GET /api/workouts/{workout_id}**
```json
Response:
{
  "workout_id": "uuid",
  "user_id": "uuid",
  "exercises": [ ... ],
  "status": "completed",
  "generated_at": "2026-02-15T10:00:00Z"
}
```

### Performance Logging Endpoints

**POST /api/performance/log**
```json
Request:
{
  "workout_id": "uuid",
  "user_id": "uuid",
  "exercises": [
    {
      "exercise_name": "Barbell Squat",
      "completed_sets": 4,
      "completed_reps": [8, 8, 7, 6],
      "weight_used": "135 lbs",
      "perceived_difficulty": 7
    }
  ],
  "notes": "Felt strong today"
}

Response:
{
  "log_id": "uuid",
  "success": true,
  "adaptation_triggered": false
}
```

**GET /api/performance/history/{user_id}**
```json
Response:
{
  "user_id": "uuid",
  "logs": [
    {
      "log_id": "uuid",
      "workout_id": "uuid",
      "logged_at": "2026-02-15T11:30:00Z",
      "completion_rate": 0.93,
      "average_difficulty": 7
    }
  ],
  "metrics": {
    "total_workouts": 15,
    "consistency_streak": 5,
    "average_completion": 0.89
  }
}
```

### Adaptation Endpoints

**GET /api/adaptation/analyze/{user_id}**
```json
Response:
{
  "user_id": "uuid",
  "recommendation": "increase_difficulty",
  "reasoning": "Consistent completion above 95% for last 3 workouts",
  "suggested_adjustments": {
    "weight_increase": "5%",
    "rep_increase": 1
  }
}
```

## 7. Database Schema Overview

### Users Table
```sql
CREATE TABLE users (
    user_id TEXT PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    name TEXT NOT NULL,
    age INTEGER,
    gender TEXT,
    fitness_level TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### UserProfiles Table
```sql
CREATE TABLE user_profiles (
    profile_id TEXT PRIMARY KEY,
    user_id TEXT NOT NULL,
    goals TEXT,  -- JSON array
    injuries TEXT,  -- JSON array
    equipment TEXT,  -- JSON array
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

### Workouts Table
```sql
CREATE TABLE workouts (
    workout_id TEXT PRIMARY KEY,
    user_id TEXT NOT NULL,
    exercises TEXT NOT NULL,  -- JSON structure
    coaching_notes TEXT,
    generated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status TEXT DEFAULT 'pending',
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

### PerformanceLogs Table
```sql
CREATE TABLE performance_logs (
    log_id TEXT PRIMARY KEY,
    workout_id TEXT NOT NULL,
    user_id TEXT NOT NULL,
    exercises_completed TEXT NOT NULL,  -- JSON structure
    completion_rate REAL,
    average_difficulty INTEGER,
    notes TEXT,
    logged_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (workout_id) REFERENCES workouts(workout_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

### Exercises Table
```sql
CREATE TABLE exercises (
    exercise_id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    category TEXT,
    muscle_groups TEXT,  -- JSON array
    equipment_required TEXT,  -- JSON array
    difficulty_level TEXT,
    contraindications TEXT,  -- JSON array (injuries to avoid)
    alternatives TEXT  -- JSON array (alternative exercise IDs)
);
```

### Relationships
- Users (1) → (Many) Workouts
- Users (1) → (Many) PerformanceLogs
- Users (1) → (1) UserProfiles
- Workouts (1) → (Many) PerformanceLogs
- Exercises (Many) → (Many) Exercises (alternatives)

## 8. AI Integration Design

### Prompt Structure

#### Workout Generation Prompt Template
```
You are an expert fitness coach. Generate a personalized workout plan.

User Context:
- Fitness Level: {fitness_level}
- Goals: {goals}
- Available Equipment: {equipment}
- Current Injuries: {injuries}
- Recent Performance: {performance_summary}

Requirements:
- Generate 5-7 exercises
- Include sets, reps, weight recommendations, and rest periods
- Avoid exercises that conflict with injuries: {injuries}
- Adjust difficulty based on recent performance trends
- Provide coaching notes explaining the workout rationale

Output Format (JSON):
{
  "exercises": [
    {
      "name": "Exercise Name",
      "sets": 4,
      "reps": 8,
      "weight": "135 lbs or bodyweight",
      "rest_seconds": 120,
      "notes": "Form cues and tips"
    }
  ],
  "coaching_notes": "Overall workout explanation"
}
```

#### Exercise Substitution Prompt Template
```
You are an expert fitness coach. Suggest a safe exercise substitution.

Context:
- Original Exercise: {exercise_name}
- User Injury: {injury}
- Available Equipment: {equipment}
- Fitness Level: {fitness_level}

Requirements:
- Suggest an alternative exercise that targets similar muscle groups
- Ensure the alternative does not aggravate the injury
- Match difficulty level to user's fitness level
- Provide reasoning for the substitution

Output Format (JSON):
{
  "original_exercise": "Exercise Name",
  "substitute_exercise": "Alternative Exercise Name",
  "reasoning": "Why this substitution is appropriate",
  "muscle_groups": ["primary", "secondary"],
  "sets": 4,
  "reps": 10,
  "notes": "Specific form cues for injury safety"
}
```

### JSON Validation

**Workout Schema:**
```python
workout_schema = {
    "type": "object",
    "required": ["exercises", "coaching_notes"],
    "properties": {
        "exercises": {
            "type": "array",
            "items": {
                "type": "object",
                "required": ["name", "sets", "reps", "rest_seconds"],
                "properties": {
                    "name": {"type": "string"},
                    "sets": {"type": "integer", "minimum": 1},
                    "reps": {"type": "integer", "minimum": 1},
                    "weight": {"type": "string"},
                    "rest_seconds": {"type": "integer", "minimum": 0},
                    "notes": {"type": "string"}
                }
            }
        },
        "coaching_notes": {"type": "string"}
    }
}
```

### Error Handling

**AI API Error Scenarios:**

1. **API Timeout**: Retry up to 3 times with exponential backoff
2. **Invalid JSON Response**: Log error, use fallback template workout
3. **Schema Validation Failure**: Attempt to repair JSON, otherwise use fallback
4. **Rate Limiting**: Queue request and retry after delay
5. **API Unavailable**: Return cached workout or generic template

**Fallback Strategy:**
- Maintain library of template workouts for each fitness level
- Use rule-based generation when AI unavailable
- Log all fallback instances for monitoring

## 9. Security Design

### Authentication Flow
1. User submits credentials
2. Backend validates against database
3. Password verified using bcrypt
4. JWT token generated with 24-hour expiration
5. Token returned to client
6. Client includes token in Authorization header for subsequent requests
7. Backend validates token on each protected endpoint

### Data Protection
- Passwords hashed with bcrypt (cost factor 12)
- JWT tokens signed with secret key
- Sensitive data never logged
- SQL queries use parameterized statements
- Input validation on all endpoints
- CORS configured for frontend origin only

### API Security
- Rate limiting on authentication endpoints (5 attempts per minute)
- Request size limits to prevent DoS
- Input sanitization to prevent injection attacks
- HTTPS required in production
- API keys for Kiro API stored in environment variables

## 10. Scalability & Future Enhancements

### Scalability Considerations

**Current Architecture (Hackathon):**
- SQLite for simplicity
- Single-instance deployment
- Synchronous AI API calls

**Production Scaling Path:**
- Migrate to PostgreSQL for concurrent access
- Implement Redis for session caching
- Add message queue (Celery) for async AI calls
- Deploy with Docker and container orchestration
- Implement database connection pooling
- Add CDN for static assets

### Future Enhancements

**Phase 1 (Post-Hackathon):**
- Mobile responsive design improvements
- Exercise video library integration
- Social features (share workouts, leaderboards)
- Advanced analytics dashboard
- Export workout history to PDF

**Phase 2 (Production):**
- Native mobile applications (iOS/Android)
- Wearable device integration (Apple Watch, Fitbit)
- Real-time form analysis using computer vision
- Nutrition tracking and meal planning
- Integration with fitness equipment (smart weights)

**Phase 3 (Advanced):**
- Multi-user coaching (trainer-client relationships)
- Marketplace for custom workout programs
- Community challenges and competitions
- AI-powered injury prediction and prevention
- Integration with health records (with consent)

### Performance Optimization

**Database Optimization:**
- Index on user_id, workout_id, logged_at columns
- Denormalize frequently accessed data
- Implement query result caching

**API Optimization:**
- Implement response caching for static data
- Use pagination for large result sets
- Compress API responses with gzip
- Batch AI requests when possible

**Frontend Optimization:**
- Lazy load components
- Implement service worker for offline capability
- Optimize bundle size with code splitting
- Cache API responses in browser storage
