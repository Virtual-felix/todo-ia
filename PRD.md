1. Product Overview [STRUCTURED_FORMAT]
- Product name: Quest
- Technical description: Modern Kanban-style task management system built with React and Go
- Marketing description: Streamline your daily tasks with an intuitive Kanban board that works the way you think
- Target audience:
  * Knowledge workers (25-45, tech-savvy, value organization)
  * Project managers (30-50, need visual task management)
  * Personal users (18+, familiar with digital tools)
- Problem solved: 
  SITUATION -> Users juggling multiple tasks across work and personal life
  PROBLEM -> Traditional to-do lists lack visual organization and workflow management
  IMPACT -> Tasks get forgotten or poorly prioritized
- Main benefits:
  * 50% faster task organization with drag-and-drop Kanban interface
  * Zero learning curve for users familiar with board-style organization
  * Real-time updates across all devices
  * Customizable workflows to match personal or team preferences

2. Technical Specifications [TECHNICAL_FORMAT]
- General architecture:
  * Frontend Layer (React SPA)
  * API Gateway Layer (Go HTTP Server)
  * Domain Layer (DDD Core)
  * Infrastructure Layer (Database, Cache)
- Technical stack:
  * Frontend: 
    - React 18.x
    - Ant Design 5.x
    - TypeScript 5.x
    - Redux Toolkit for state management
    - React Query for data fetching
  * Backend: 
    - Go 1.24.x
    - Chi router for HTTP routing
    - PGX for database operations
    - go-playground/validator for validation
  * Database: PostgreSQL 15.x (for ACID compliance and JSON support)
  * Infrastructure:
    - Docker for containerization
    - Redis for caching
    - Prometheus for monitoring

3. Data Model [JSON_FORMAT]
- Main entities:
  ```json
  {
    "List": {
      "attributes": [
        "id",
        "name",
        "is_system",
        "position",
        "created_at",
        "updated_at"
      ],
      "relations": ["has_many:Tasks"],
      "constraints": [
        "name:required,max=50",
        "is_system:boolean",
        "position:required,numeric"
      ]
    },
    "Task": {
      "attributes": [
        "id",
        "title",
        "description",
        "list_id",
        "status",
        "due_date",
        "created_at",
        "completed_at",
        "archived_at",
        "original_list_id",
        "repeat_type",
        "repeat_value",
        "repeat_frequency",
        "position"
      ],
      "relations": ["belongs_to:List", "has_many:Labels"],
      "constraints": [
        "title:required,max=100",
        "description:markdown",
        "list_id:required,exists:lists,id",
        "original_list_id:exists:lists,id",
        "status:in(todo,done,archived)",
        "repeat_type:in(none,after_completion,fixed_schedule)",
        "repeat_value:numeric",
        "repeat_frequency:in(hours,days,weeks,months,years)"
      ]
    },
    "Label": {
      "attributes": ["id", "name", "color", "created_at"],
      "relations": ["belongs_to:Task"],
      "constraints": ["name:required,max=20", "color:required,hex"]
    }
  }
  ```

4. API and Access Points [REST_FORMAT]
- Endpoints:
  ```
  # Lists
  GET    /api/v1/lists              : List all lists
  POST   /api/v1/lists              : Create new list {
                                      name*,
                                      position
                                     }
  GET    /api/v1/lists/:id          : Get list details with tasks
  PUT    /api/v1/lists/:id          : Update list {name, position}
  DELETE /api/v1/lists/:id          : Delete list (blocked for system lists)

  # Tasks
  GET    /api/v1/tasks              : List all tasks with optional filters
                                     ?list_id=uuid
                                     ?status=todo|done|archived
                                     ?due_before=date
                                     ?due_after=date
                                     ?completed_before=date
                                     ?completed_after=date
                                     ?original_list_id=uuid
  POST   /api/v1/lists/:id/tasks    : Create task in list {
                                      title*,
                                      description,
                                      due_date,
                                      repeat_type,
                                      repeat_value,
                                      repeat_frequency
                                     }
  GET    /api/v1/tasks/:id          : Get task details
  PUT    /api/v1/tasks/:id          : Update task details
  PATCH  /api/v1/tasks/:id/status   : Update task status {status: done}
  PATCH  /api/v1/tasks/:id/position : Update task position {position, list_id}
  DELETE /api/v1/tasks/:id          : Delete task

  # Labels
  GET    /api/v1/labels            : List all labels
  POST   /api/v1/labels            : Create label {name, color}
  PUT    /api/v1/labels/:id        : Update label
  DELETE /api/v1/labels/:id        : Delete label

  # Task Labels
  POST   /api/v1/tasks/:id/labels  : Add label to task
  DELETE /api/v1/tasks/:id/labels/:label_id : Remove label from task
  ```
- Authentication: JWT tokens (72h expiry, refresh token 90 days)
- Rate limiting: 100 requests per minute per user

5. User Interface [COMPONENTS_FORMAT]
- Main page:
  * QuestView:
    - Components:
      * MainLayout (Ant Design Layout)
        - TabsContainer (Ant Design Tabs)
          * MainTab (Default tab - Fixed layout)
            - TopGrid (2-column grid)
              * DailyList (Left column)
                - AutoArchive: 2:00 AM daily
              * WeeklyList (Right column)
                - AutoArchive: Monday 2:00 AM
            - BacklogList (Full width)
          * CustomListTabs (Dynamic tabs)
            - SingleListView
          * HistoryTab (Read-only)
            - CompletedTasksList
            - FilterByOriginalList
            - SearchByDate
        - AddTabButton (Ant Design Button + Modal)
          * Plus icon
          * "New List" modal with name input
      * ListComponent (Ant Design List + Custom)
        - TaskList
        - CreateTaskButton
        - ListHeader
      * TaskCard (Ant Design Card + Custom)
        - Checkbox for completion
        - Grayed out style when done
        - Progress indicators:
          * DueDate Progress Bar (all tasks with due_date)
            - Shows time remaining until due date
            - Color coding:
              > Green: >3 days remaining
              > Orange: 1-3 days remaining
              > Red: <24h remaining or overdue
            - Tooltip showing exact time remaining
            - Updates in real-time
          * Recurrence Progress Bar (completed recurring tasks only)
            - Shows time until next occurrence
            - Based on completed_at + recurrence settings
            - Simple progress indicator (no color coding)
            - Tooltip showing next occurrence date/time
            - Only visible after task completion
            - Disappears when task becomes active again
      * TaskDetailDrawer (Ant Design Drawer)
      * MarkdownEditor (React-SimpleMDE)
      * HistoryView
        - CompletedTaskCard (Read-only variant)
        - FilterBar
        - DateRangePicker
    - States:
      * Loading
      * DraggingTask
      * Filtering
      * Creating/Editing Task
      * Creating New List
      * Error
      * Completing Task
    - User actions:
      * Main Tab (System Lists)
        - View daily and weekly lists side by side
        - View backlog list below
        - Create/Edit/Delete tasks
        - Drag tasks between lists
      * Custom Lists
        - Create new list via "+" button
        - Each custom list gets its own tab
        - Close/Delete custom list tabs
        - Create/Edit/Delete tasks in custom lists
      * General Actions
        - Mark task as done
        - Set task recurrence
        - Add/Remove labels
        - Filter tasks by status/labels
    - Displayed data:
      * Main Tab Layout
        - Daily and Weekly tasks in split view
        - Backlog tasks in full width below
      * Custom List Tabs
        - One list per tab
        - List name in tab header
      * Task details
        - Title
        - Markdown description
        - Due date
        - Recurrence pattern
        - Completion status
        - Labels
        - Task Progress
          * Due Date Progress (when due_date is set)
            - Time remaining until due date
            - Color-coded status indicator
            - Exact due date/time in tooltip
          * Recurrence Progress (for completed recurring tasks)
            - Time until next occurrence
            - Simple progress indicator
            - Next occurrence date/time in tooltip
      * Statistics
        - Tasks due today
        - Completed vs pending
        - Tasks by list
        - Upcoming recurring tasks

6. Testing and Quality [METRICS_FORMAT]
- Minimum test coverage:
  * Backend: 80% code coverage
  * Frontend: 70% code coverage
  * E2E tests: Critical user paths covered
- Maximum response time:
  * API requests: 200ms
  * Database queries: 100ms
  * Frontend rendering: 50ms First Contentful Paint
- Supported load:
  * 1000 concurrent users per instance
  * 100 requests per second per instance
- Expected availability: 99.9% uptime

7. Security [CHECKLIST_FORMAT]
- Authentication:
  * JWT-based authentication
  * Secure password hashing with bcrypt (cost factor 12)
  * Rate limiting on login attempts (5 per 15 minutes)
  * CSRF protection for web forms
- Authorization:
  * Role-based access control (RBAC)
  * Board-level permissions (Owner, Editor, Viewer)
  * Resource-level access validation
  * API endpoint permission checks
- Data validation:
  * Input sanitization on all user inputs
  * Strong parameter validation
  * JSON schema validation for API requests
  * XSS prevention
- Encryption:
  * TLS 1.3 for all communications
  * AES-256 for sensitive data at rest
  * Secure key management using environment variables
  * Regular security audits

8. Monitoring [ALERTS_FORMAT]
- Critical metrics:
  * API Response Time: [<200ms, >500ms]
  * Error Rate: [<0.1%, >1%]
  * CPU Usage: [<70%, >85%]
  * Memory Usage: [<75%, >90%]
  * Database Connections: [<80%, >90%]
  * HTTP 5xx Errors: [<10 per hour, >50 per hour]
- Required logs:
  * Format: JSON structured logging
  * Fields: timestamp, service, level, message, trace_id, user_id
  * Retention: 30 days hot storage, 1 year cold storage
  * Analysis: ELK Stack (Elasticsearch, Logstash, Kibana)

9. Deployment Plan [TIMELINE_FORMAT]
- Phase 1 - MVP (30 days):
  * Features:
    - Basic board creation and management
    - List and task CRUD operations
    - Drag and drop functionality
    - Basic user authentication
    - Mobile-responsive design
  * Timeline:
    - Days 1-5: Initial project setup and architecture
    - Days 6-15: Core backend implementation
    - Days 16-25: Frontend development
    - Days 26-30: Testing and bug fixes
  * Validation criteria:
    - All unit tests passing
    - < 500ms average API response time
    - Successful task operations on multiple devices
    - Zero critical security issues

10. Documentation [STRUCTURE_FORMAT]
- API:
  * OpenAPI 3.0 specification (docs/api.yaml)
    - Comprehensive API schema definitions
    - Full endpoint documentation with examples
    - Authentication and security specifications
    - Response/Request models
  * Interactive Swagger UI documentation at /swagger
  * Postman collection auto-generated from OpenAPI spec
  * API versioning via Accept header and URL path
- Code:
  * Go: godoc format with examples
  * React: JSDoc with component stories
  * TypeScript: Detailed type definitions
  * SQL: Database schema documentation
- Architecture:
  * System context diagram
  * Container diagram (C4 model)
  * Component diagram
  * Domain model diagram
- User:
  * Getting started guide
  * Feature documentation
  * Troubleshooting guide
  * FAQ section
