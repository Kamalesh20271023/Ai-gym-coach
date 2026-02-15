# AI Gym Coach – Software Requirements Specification

## 1. Project Overview

AI Gym Coach is a modular AI-driven adaptive fitness system that generates personalized workout plans, adapts to user performance, handles injury constraints, and provides intelligent coaching feedback through a closed-loop adaptive system.

## 2. Problem Statement

Traditional fitness applications provide static workout plans that fail to adapt to individual progress, injuries, or performance variations. Users need an intelligent system that dynamically adjusts workouts based on real-time performance data and personal constraints.

## 3. Objectives

- Deliver personalized workout plans tailored to user fitness levels and goals
- Implement adaptive progression that responds to performance metrics
- Provide injury-aware exercise substitution to ensure safe training
- Enable performance trend analysis for data-driven insights
- Integrate AI-powered coaching explanations for user guidance
- Create a closed feedback loop for continuous workout optimization

## 4. Scope of the System

### In Scope
- User profile management with fitness goals and injury tracking
- AI-powered workout generation using Kiro API
- Performance logging and historical data storage
- Adaptive workout progression based on logged performance
- Injury-aware exercise substitution engine
- Performance trend visualization and analysis
- Structured JSON-based workout output
- RESTful API backend with FastAPI
- React-based web frontend
- SQLite database for data persistence

### Out of Scope
- Mobile native applications (iOS/Android)
- Video exercise demonstrations
- Social features and community interaction
- Payment processing and subscription management
- Wearable device integration
- Real-time video form analysis

## 5. Functional Requirements

### 5.1 User Management

**FR-1.1**: The system shall allow users to create accounts with email and password.

**FR-1.2**: The system shall store user profiles including:
- Name, age, gender
- Fitness level (beginner, intermediate, advanced)
- Fitness goals (strength, endurance, weight loss, muscle gain)
- Current injuries or physical limitations
- Equipment availability

**FR-1.3**: The system shall allow users to update their profile information at any time.

**FR-1.4**: The system shall authenticate users securely before granting access to personalized features.

### 5.2 Workout Generation

**FR-2.1**: The system shall generate personalized workout plans based on:
- User fitness level
- User goals
- Available equipment
- Current injuries
- Historical performance data

**FR-2.2**: The system shall use Kiro API to generate workouts with structured JSON output containing:
- Exercise name
- Sets and reps
- Rest periods
- Intensity level
- Exercise notes

**FR-2.3**: The system shall validate AI-generated workout JSON against a predefined schema.

**FR-2.4**: The system shall provide AI-generated coaching explanations for workout rationale.

**FR-2.5**: The system shall allow users to request new workout generation on demand.

### 5.3 Adaptive Progression

**FR-3.1**: The system shall analyze performance logs to determine progression readiness.

**FR-3.2**: The system shall automatically increase workout difficulty when:
- User consistently completes prescribed sets/reps
- Performance metrics show improvement trends
- User reports workouts as "too easy"

**FR-3.3**: The system shall decrease workout difficulty when:
- User fails to complete prescribed sets/reps multiple times
- Performance metrics show declining trends
- User reports excessive fatigue

**FR-3.4**: The system shall adjust progression parameters including:
- Weight/resistance
- Repetitions
- Sets
- Exercise complexity
- Rest periods

### 5.4 Injury Handling

**FR-4.1**: The system shall maintain a database of exercises with associated muscle groups and movement patterns.

**FR-4.2**: The system shall identify exercises that conflict with reported injuries.

**FR-4.3**: The system shall automatically substitute conflicting exercises with safe alternatives.

**FR-4.4**: The system shall use AI to suggest injury-appropriate exercise modifications.

**FR-4.5**: The system shall allow users to report new injuries and update injury status.

### 5.5 Performance Logging

**FR-5.1**: The system shall allow users to log completed workouts with:
- Actual sets and reps completed
- Weight used
- Perceived difficulty (1-10 scale)
- Notes and comments

**FR-5.2**: The system shall timestamp all performance logs.

**FR-5.3**: The system shall calculate performance metrics including:
- Completion rate
- Volume progression
- Consistency streaks

**FR-5.4**: The system shall store historical performance data for trend analysis.

### 5.6 AI Integration

**FR-6.1**: The system shall integrate with Kiro API for:
- Workout generation
- Exercise substitution recommendations
- Coaching explanations
- Progression analysis

**FR-6.2**: The system shall construct AI prompts with:
- User profile context
- Performance history
- Current constraints
- Specific generation requirements

**FR-6.3**: The system shall handle AI API errors gracefully with fallback mechanisms.

**FR-6.4**: The system shall validate and sanitize AI responses before storage.

## 6. Non-Functional Requirements

### 6.1 Scalability

**NFR-1.1**: The system shall support up to 1000 concurrent users during hackathon demonstration.

**NFR-1.2**: The database schema shall be designed to accommodate future scaling to PostgreSQL.

**NFR-1.3**: The API shall be stateless to enable horizontal scaling.

### 6.2 Security

**NFR-2.1**: The system shall hash passwords using bcrypt or equivalent.

**NFR-2.2**: The system shall implement JWT-based authentication.

**NFR-2.3**: The system shall validate and sanitize all user inputs.

**NFR-2.4**: The system shall protect against SQL injection attacks.

**NFR-2.5**: The system shall implement CORS policies for frontend-backend communication.

### 6.3 Performance

**NFR-3.1**: Workout generation shall complete within 10 seconds.

**NFR-3.2**: API endpoints shall respond within 2 seconds for standard operations.

**NFR-3.3**: The frontend shall render workout displays within 1 second.

**NFR-3.4**: Database queries shall be optimized with appropriate indexing.

### 6.4 Maintainability

**NFR-4.1**: The codebase shall follow PEP 8 style guidelines for Python.

**NFR-4.2**: The codebase shall follow ESLint standards for React/JavaScript.

**NFR-4.3**: The system shall use modular architecture with clear separation of concerns.

**NFR-4.4**: The system shall include inline documentation for complex logic.

**NFR-4.5**: The API shall be documented using OpenAPI/Swagger specification.

### 6.5 Usability

**NFR-5.1**: The user interface shall be intuitive and require minimal training.

**NFR-5.2**: The system shall provide clear error messages for user actions.

**NFR-5.3**: The system shall be responsive and work on desktop and tablet devices.

**NFR-5.4**: The system shall provide loading indicators for asynchronous operations.

## 7. System Constraints

**C-1**: The system must use SQLite as the database for hackathon simplicity.

**C-2**: The system must integrate with Kiro API for AI functionality.

**C-3**: The system must be deployable within hackathon timeframe (24-48 hours).

**C-4**: The system must run on standard development machines without specialized hardware.

**C-5**: The frontend must be web-based using React framework.

**C-6**: The backend must use FastAPI framework with Python.

## 8. Assumptions & Dependencies

### Assumptions

**A-1**: Users have basic understanding of fitness terminology.

**A-2**: Users will provide accurate information about injuries and fitness levels.

**A-3**: Users have internet connectivity for AI API calls.

**A-4**: Kiro API will be available and responsive during system operation.

### Dependencies

**D-1**: Kiro API for AI-powered workout generation and coaching.

**D-2**: React library and ecosystem for frontend development.

**D-3**: FastAPI framework for backend API development.

**D-4**: SQLite database engine.

**D-5**: Python 3.8+ runtime environment.

**D-6**: Node.js and npm for frontend build tooling.

## 9. Success Metrics

**M-1**: System successfully generates personalized workouts for 100% of user profiles.

**M-2**: Adaptive progression correctly adjusts difficulty based on performance logs.

**M-3**: Injury-aware substitution prevents conflicting exercises in 100% of cases.

**M-4**: AI integration produces valid JSON workout structures in 95%+ of requests.

**M-5**: System demonstrates complete closed-loop adaptation during hackathon demo.

**M-6**: User can complete full workflow (signup → workout generation → logging → adaptation) in under 5 minutes.

**M-7**: System handles concurrent demo users without performance degradation.

**M-8**: Zero critical security vulnerabilities in authentication and data handling.
