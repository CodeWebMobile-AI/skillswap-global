# skillswap-global Architecture

## Overview
To connect individuals globally for reciprocal skill-sharing and collaborative project completion, fostering cross-cultural learning and mutual benefit.

### Problem Statement
Individuals struggle to find accessible and trustworthy avenues for acquiring new skills, particularly those requiring hands-on experience or personalized mentorship, while also lacking opportunities to apply their existing skills in meaningful projects that contribute to their personal or professional development.

### Target Audience
Remote workers, career changers, hobbyists, and lifelong learners of all ages and backgrounds seeking to acquire new skills, practice existing ones, and collaborate on projects with individuals from diverse cultural and professional backgrounds.

### Core Entities
Users, Skills, Projects, Swaps (Skill Exchange Agreements), Reviews, Languages

## Design System

### Typography
- **Font**: Open Sans
- **Font Stack**: `'Open Sans', sans-serif`
- **Google Fonts**: https://fonts.google.com/specimen/Open+Sans
- **Rationale**: Open Sans is a highly readable and versatile sans-serif font, well-suited for body text and UI elements. It's widely used and known for its clarity at various sizes, ensuring good accessibility. [Source: Google Fonts, established design practices]

### Color Palette
A complementary palette chosen for its balance and professional feel. Blue and orange create a visually appealing contrast, while gray provides a neutral background.

- **Primary**: `#29ABE2` - Main brand color, used for headers and primary actions.
- **Secondary**: `#F26522` - Supporting color for secondary elements.
- **Accent**: `#FFDA63` - Used for call-to-action buttons and highlights.
- **Neutral Text**: `#333333` - Primary text color for high contrast and readability.
- **Neutral Background**: `#F7F7F7` - Main background color for content areas.
- **Neutral Border**: `#DDDDDD` - For card borders, dividers, and form inputs.
- **Success**: `#4CAF50` - For success messages and confirmation.
- **Warning**: `#FF9800` - For warnings and non-critical alerts.
- **Danger**: `#F44336` - For error messages and destructive actions.

## System Architecture

### 1. Core Components & Rationale
The architecture is built around a modular and scalable approach. The backend is a Laravel API, serving a React frontend built with Inertia.js and TypeScript. Redis is used for real-time functionality. Justification: This combination leverages Laravel's robust features and ecosystem with React's component-based UI, and Inertia.js simplifies the communication between the two. TypeScript adds type safety and improves code maintainability. Redis offers a fast and reliable pub/sub mechanism for real-time updates. [Source: Laravel Documentation, React Documentation, Inertia.js Documentation, Redis Documentation]

### 2. Database Schema Design
The database schema will include the following tables:

*   `users`: id, name, email, password, profile_picture, bio, location, language, verification_status, created_at, updated_at
*   `skills`: id, name, description, created_at, updated_at
*   `user_skills`: user_id (FK), skill_id (FK), level, created_at, updated_at
*   `projects`: id, user_id (FK), title, description, status, created_at, updated_at
*   `project_skills`: project_id (FK), skill_id (FK), required_level, created_at, updated_at
*   `swaps`: id, initiator_user_id (FK), receiver_user_id (FK), project_id (FK), status, created_at, updated_at
*   `swap_skills`: swap_id (FK), skill_id (FK), provided_by, level, created_at, updated_at
*   `reviews`: id, swap_id (FK), reviewer_user_id (FK), reviewee_user_id (FK), rating, comment, created_at, updated_at
*   `languages`: id, name, code, created_at, updated_at
*   `user_languages`: user_id (FK), language_id (FK), proficiency, created_at, updated_at

Relationships are defined using Laravel's Eloquent ORM. Indexes are crucial for performance. [Source: Laravel Eloquent Documentation, Database Design Best Practices]

### 3. API Design & Key Endpoints
A RESTful API using Laravel's resource controllers and API resources. API versioning (e.g., `/api/v1/`) is essential. Authentication will be handled via Sanctum for SPA authentication and Passport for OAuth (potentially for third-party integrations later). Key endpoints include:

*   `/api/v1/users`: CRUD operations for users.
*   `/api/v1/skills`: CRUD operations for skills.
*   `/api/v1/projects`: CRUD operations for projects.
*   `/api/v1/swaps`: CRUD operations for swaps.
*   `/api/v1/reviews`: CRUD operations for reviews.
*   `/api/v1/auth/login`: User login.
*   `/api/v1/auth/register`: User registration.
*   `/api/v1/auth/logout`: User logout.
*   `/api/v1/auth/me`: Get authenticated user.

Rate limiting is implemented using Laravel's built-in mechanisms. Input validation is enforced using Laravel's validation rules. [Source: Laravel Routing Documentation, Laravel API Resources Documentation, Laravel Sanctum Documentation, Laravel Passport Documentation]

### 4. Frontend Structure (TypeScript)
The frontend is structured with React components, using TypeScript for type safety. A common directory structure includes:

*   `Components/`: Reusable UI components (e.g., buttons, forms, cards).
*   `Pages/`: Page-level components (e.g., Home, Profile, ProjectDetails).
*   `Layouts/`: Layout components (e.g., MainLayout, AuthLayout).
*   `Hooks/`: Custom React hooks for managing state and logic.
*   `Types/`: TypeScript type definitions for API responses and component props.
*   `Services/`: API client (using Axios or similar) for making requests to the backend.
*   `Utils/`: Utility functions (e.g., date formatting, string manipulation).

State management will be handled using React Context API or Zustand for simplicity. Global state can be managed with Redux or Recoil for larger-scale state management if the application complexity increases. [Source: React Component Structure Best Practices, TypeScript Best Practices]

### 5. Real-time & Events Architecture
Redis is used as the message broker, and `laravel-echo-server` provides a WebSocket server. Laravel events are broadcast via Redis channels.  Frontend components subscribe to these channels using `laravel-echo`.  For example, a `NewMessage` event can be broadcast when a new chat message is created. The frontend then listens to this event and updates the chat UI. Private channels are used for user-specific events (e.g., notifications). Authentication for private channels is handled by Laravel Echo's authentication endpoint. [Source: Laravel Broadcasting Documentation, laravel-echo-server Documentation, Redis Pub/Sub Documentation]

### 6. Authentication & Authorization Flow
Sanctum is used for authentication, providing SPA authentication via API tokens.  When a user logs in, Sanctum issues a token. This token is stored in the browser's local storage or cookies (with appropriate security attributes). The frontend then includes this token in the `Authorization` header of subsequent API requests.  Authorization is handled using Laravel's policies.  Policies define which users are allowed to perform specific actions on specific resources.  For example, a policy might define that only the owner of a project can edit it.  Middleware is used to protect API endpoints based on these policies. [Source: Laravel Sanctum Documentation, Laravel Authorization Documentation]

### 7. Deployment & Scalability Plan
The application will be deployed on a cloud platform like AWS, Google Cloud, or DigitalOcean. Docker is used for containerization to ensure consistent environments. Database server (e.g., MySQL, PostgreSQL) can be on a separate instance or using managed database services. Redis will be deployed separately as well, potentially using a managed Redis service for scalability. Load balancing is implemented to distribute traffic across multiple application instances. Horizontal scaling is achieved by adding more application instances behind the load balancer. Caching mechanisms (e.g., Redis caching, HTTP caching) are employed to reduce database load. A CI/CD pipeline using GitHub Actions or similar is set up for automated deployments. [Source: Docker Documentation, Cloud Provider Documentation, CI/CD Best Practices]

### 8. Testing Strategy
A comprehensive testing strategy includes:

*   **Backend:** Pest for unit and feature tests.  Tests cover API endpoints, business logic, and database interactions.
*   **Frontend:** Vitest and React Testing Library (RTL) for unit and component tests. Tests cover UI components, state management, and API integrations.
*   **End-to-end (E2E) tests:** Playwright or Cypress for testing the entire application flow.  These tests simulate user interactions and verify that the application functions correctly from end to end.

Test Driven Development (TDD) is encouraged to ensure high test coverage. Automated testing is integrated into the CI/CD pipeline. [Source: Pest Documentation, Vitest Documentation, React Testing Library Documentation, Playwright Documentation, Cypress Documentation]

### 9. Security & Hardening Plan
A 'security-first' approach addressing OWASP Top 10 vulnerabilities:

*   **Input Validation:** Laravel's validation rules and middleware are used to validate all user inputs.
*   **Output Encoding:** Data is properly encoded before being displayed to prevent XSS attacks.
*   **Authentication & Authorization:** Sanctum and Laravel's policies are used for secure authentication and authorization.
*   **Password Hashing:** Bcrypt hashing is used for password storage.
*   **SQL Injection Prevention:** Eloquent ORM is used to prevent SQL injection attacks.
*   **CSRF Protection:** Laravel's CSRF protection middleware is enabled.
*   **Rate Limiting:** Laravel's rate limiting is used to prevent brute-force attacks.
*   **Regular Security Audits:** Regular security audits are conducted to identify and address vulnerabilities.
*   **Dependency Management:** Dependencies are regularly updated to patch security vulnerabilities using Composer and NPM.
*   **HTTPS:** The application is served over HTTPS to encrypt traffic.

Content Security Policy (CSP) is implemented to prevent XSS attacks. [Source: OWASP Top 10, Laravel Security Documentation]

### 10. Logging & Observability
Laravel's logging facilities are used to log application events. A dedicated `stderr` channel is configured to send logs to standard error, which can be collected by a log management system (e.g., ELK stack, Graylog).  Structured logging (e.g., JSON format) is used for easier parsing and analysis.  Monitoring tools (e.g., Prometheus, Grafana) are used to monitor key metrics such as CPU usage, memory usage, response time, and error rate.  Alerts are configured to notify administrators of critical issues. Distributed tracing (e.g., Jaeger, Zipkin) is used to track requests across different services. [Source: Laravel Logging Documentation, Prometheus Documentation, Grafana Documentation]

## Feature Implementation Roadmap

### 1. Skill profile creation with verification options
Users can create profiles, listing their skills and experience. Verification options (e.g., email, ID verification) are available.

**Database Changes:**
- Add 'verification_status' column to the 'users' table (enum: pending, verified, rejected).
- Potentially add a 'user_verifications' table to store verification documents and details.

**Backend Components:**
- Update 'UserController' to handle profile updates and verification requests.
- Create a 'VerificationService' class to manage verification processes.

**Frontend Components:**
- Update 'ProfileForm.tsx' to include skill input and verification options.
- Create a 'VerificationModal.tsx' component to guide users through the verification process.
- Update `Pages/Profile/Edit.tsx` to show Verification status.

**API Endpoints:**
- PUT /api/v1/users/{id}
- POST /api/v1/users/{id}/request-verification
- GET /api/v1/users/{id}/verification-status

**Testing Requirements:**
- Feature test for 'UserController' to ensure profile updates are handled correctly.
- Unit test for 'VerificationService' to ensure verification requests are processed correctly.
- Component test for 'ProfileForm.tsx' to ensure skill input and verification options are displayed correctly.
- E2E test verifying user profile editing and verification process.


### 2. Project posting and browsing functionality
Users can post projects with descriptions, required skills, and collaborate with others.

**Database Changes:**
- Update 'projects' table to include columns for project status (e.g., open, in progress, completed), start date, and end date (optional).
- Add indexes on relevant columns like status, skills, and location.

**Backend Components:**
- Create 'ProjectController' for project CRUD operations.
- Create 'ProjectSearchService' to handle project searching and filtering.
- Update 'project_skills' table (Many to Many) with level of skill to allow fine tuned searches

**Frontend Components:**
- Create 'ProjectCard.tsx' component to display project summaries.
- Create 'ProjectForm.tsx' component for project creation and editing.
- Create `Pages/Projects/Index.tsx` for showing project listings
- Create `Pages/Projects/Show.tsx` for showing project details

**API Endpoints:**
- GET /api/v1/projects
- POST /api/v1/projects
- GET /api/v1/projects/{id}
- PUT /api/v1/projects/{id}
- DELETE /api/v1/projects/{id}

**Testing Requirements:**
- Feature test for 'ProjectController' to ensure project CRUD operations are handled correctly.
- Unit test for 'ProjectSearchService' to ensure project searching and filtering work as expected.
- Component test for 'ProjectCard.tsx' and 'ProjectForm.tsx' to ensure they render correctly and handle user input.
- E2E tests for project creation and browsing.


### 3. Matching algorithm for identifying suitable skill swaps and project collaborators
An algorithm that matches users with compatible skills and projects, suggesting potential skill swaps and collaborations.

**Backend Components:**
- Create a 'MatchingService' class that implements the matching algorithm. This service will query the database to find users with matching skills and projects.  The algorithm can be based on skill levels, location, languages and project needs.
- Use Laravel's Queues to handle the algorithm processing in the background to not overload the frontend.

**Frontend Components:**
- Create a 'MatchSuggestions.tsx' component to display the suggested skill swaps and project collaborators.
- Update 'ProfilePage.tsx' to show the relevant MatchSuggestions.

**API Endpoints:**
- GET /api/v1/matches/users/{user_id}
- GET /api/v1/matches/projects/{project_id}

**Testing Requirements:**
- Unit test for 'MatchingService' to ensure the matching algorithm works correctly and returns the expected results.
- Feature Test verifying backend data is displayed in the front end (data display only).
- End-to-End tests to verify Matches are displayed in a User profile.


### 4. Secure communication and collaboration tools (video call integration, chat, file sharing)
Secure communication channels including real-time chat, video conferencing integration, and file sharing capabilities within projects.

**Database Changes:**
- Create 'messages' table: id, sender_id (FK), receiver_id (FK), content, created_at, updated_at, attachment_url (nullable).
- Create 'rooms' table: id, type (direct, project), related_id (user_id or project_id), created_at, updated_at.
- Create 'room_users' table (user_id, room_id)

**Backend Components:**
- Create 'MessageController' for handling messages.
- Integrate a video conferencing API (e.g., Twilio, Zoom API) to handle video calls.  Store any needed metadata to our DB tables. 
- Implement file upload and storage functionality (e.g., using AWS S3 or similar).  Store any needed metadata to our DB tables.
- Add policies to ensure Users are allowed to communicate in the proper contexts (project, direct, etc.).

**Frontend Components:**
- Create 'ChatWindow.tsx' component for real-time chat.
- Implement UI for initiating and joining video calls (using the chosen video conferencing API's SDK).
- Implement UI for file uploading and downloading.

**API Endpoints:**
- GET /api/v1/messages/{room_id}
- POST /api/v1/messages/{room_id}
- POST /api/v1/files/upload
- GET /api/v1/files/{file_id}/download

**Real-time Events:**
- Broadcast 'NewMessage' event on a private channel: 'rooms.{room_id}' when a new message is sent.

**Testing Requirements:**
- Feature test for 'MessageController' to ensure messages are sent and received correctly.
- Component test for 'ChatWindow.tsx' to ensure it displays messages in real time.
- Integration tests verifying video call initiation and file upload functionalities.
- E2E test validating secure communication and file sharing flow.


### 5. Review and rating system for skill swaps and project contributions
Users can review and rate skill swaps and project contributions, providing feedback and building a reputation system.

**Database Changes:**
- The 'reviews' table has been defined in the foundational architecture.
- Consider adding an 'anonymous' boolean field to `reviews` table.

**Backend Components:**
- Create 'ReviewController' for handling review creation, retrieval, and updates.
- Implement logic to calculate average ratings and display them on user profiles and project pages.
- Add policies to ensure users can only review swaps they participated in.

**Frontend Components:**
- Create 'ReviewForm.tsx' component for submitting reviews.
- Create 'RatingDisplay.tsx' component to display average ratings and individual reviews.
- Implement UI to show user Reputation Score (calculated from reviews).

**API Endpoints:**
- GET /api/v1/reviews/{swap_id}
- POST /api/v1/reviews/{swap_id}
- GET /api/v1/users/{user_id}/reviews

**Testing Requirements:**
- Feature test for 'ReviewController' to ensure reviews are created and associated correctly.
- Unit test for rating calculation logic.
- Component test for 'ReviewForm.tsx' and 'RatingDisplay.tsx' to ensure they render correctly.
- E2E tests for review submission and reputation display flow.


### 6. Multilingual support for global accessibility
The application supports multiple languages, allowing users from different countries to use the platform in their native language.

**Database Changes:**
- The 'languages' and 'user_languages' tables are defined in the foundational architecture.
- Implement database schema for storing translated content (e.g., a separate table for translations or a JSON column on existing tables).  Consider using a package like astrotomic/laravel-translatable to simplify this.

**Backend Components:**
- Integrate a translation library (e.g., `astrotomic/laravel-translatable`) to handle translations.
- Create middleware to detect the user's preferred language and set the application locale accordingly.
- Add logic to store and retrieve translated content from the database.

**Frontend Components:**
- Implement a language switcher component in the UI.
- Use i18n library (e.g., `react-i18next`) to translate UI elements.
- Use JS and CSS libraries to support RTL languages (if required).

**Testing Requirements:**
- Integration tests to verify that translations are loaded and displayed correctly.
- Component tests for the language switcher component.
- E2E tests verifying the multilingual support throughout the application.


## Metadata
- **Generated**: 2025-06-18T08:28:02.533289+00:00
- **Project Type**: Laravel React Starter Kit
- **Architecture Version**: 1.0.0

---
*This architecture document is maintained by the CodeWebMobile AI system and should be the source of truth for all development decisions.*
