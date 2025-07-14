1. Product Overview [STRUCTURED_FORMAT]
- Product name: Quest
- Technical description: Modern task management system with Kanban-inspired interface built with React and Go
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
  * 50% faster task organization with intuitive list-based interface
  * Zero learning curve for users familiar with task management tools
  * Real-time updates across all devices
  * Flexible workflows with daily/weekly organization

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
    "User": {
      "attributes": [
        "id",
        "email",
        "name",
        "created_at",
        "updated_at"
      ],
      "relations": ["has_many:Lists", "has_many:Tasks"],
      "constraints": [
        "email:required,unique,email",
        "name:required,max=100"
      ]
    },
    "List": {
      "attributes": [
        "id",
        "name",
        "is_system",
        "system_type",
        "position",
        "user_id",
        "created_at",
        "updated_at"
      ],
      "relations": ["has_many:Tasks", "belongs_to:User"],
      "constraints": [
        "name:required,max=50",
        "is_system:boolean",
        "system_type:in(daily,weekly,backlog,history,custom)",
        "user_id:required,exists:users,id",
        "position:required,numeric"
      ]
    },
    "Task": {
      "attributes": [
        "id",
        "title",
        "description",
        "list_id",
        "user_id",
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
      "relations": ["belongs_to:List", "belongs_to:User", "many_to_many:Labels"],
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
    "TaskLabel": {
      "attributes": [
        "task_id",
        "label_id",
        "created_at"
      ],
      "relations": ["belongs_to:Task", "belongs_to:Label"],
      "constraints": [
        "task_id:required,exists:tasks,id",
        "label_id:required,exists:labels,id"
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
  POST   /api/v1/tasks              : Create task {
                                      title*,
                                      description,
                                      list_id*,
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

9. Documentation [STRUCTURE_FORMAT]
- API:
  * OpenAPI 3.0 specification (docs/api.yaml)
    - Comprehensive API schema definitions
    - Full endpoint documentation with examples
    - Authentication and security specifications
    - Response/Request models
  * Interactive Swagger UI documentation at /swagger
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

10. User Experience Specifications [UX_FORMAT]
- Onboarding flow:
  * First-time user experience:
    - Welcome tour highlighting key features
    - Interactive tutorial for Kanban basics
    - Sample tasks and lists pre-populated
    - Customization wizard for workspace setup
  * Account setup guidance:
    - Email verification process
    - Profile completion checklist
    - Workspace preferences configuration
    - Team invitation flow (if applicable)
- Empty states:
  * Lists:
    - Helpful message: "Start organizing by adding your first task"
    - Quick-add button prominently displayed
    - Suggested task templates
    - Import options from other tools
  * Boards:
    - Welcome message with getting started tips
    - Sample board templates
    - Quick setup wizard
- Error handling:
  * User-facing error messages:
    - Clear, non-technical language
    - Actionable recovery steps
    - Contact support option
    - Error tracking ID (hidden, expandable)
  * Common scenarios:
    - Network connectivity issues
    - Invalid input handling
    - Permission denied states
    - Concurrent edit conflicts
- Keyboard shortcuts:
  * Global shortcuts:
    - New task: Cmd/Ctrl + N
    - Quick search: Cmd/Ctrl + K
    - Save changes: Cmd/Ctrl + S
    - Close modal: Esc
  * List management:
    - Navigate lists: Alt + Arrow keys
    - Create list: Cmd/Ctrl + Shift + L
    - Delete list: Cmd/Ctrl + Shift + D
  * Task management:
    - Mark complete: Space
    - Edit task: Enter
    - Delete task: Delete
    - Move task: Shift + Arrow keys
- Notification system:
  * Email notifications:
    - Daily digest option
    - Due date reminders
    - Assignment notifications
    - Weekly summary reports
  * Push notifications (mobile):
    - Due date alerts
    - Task assignments
    - Mention notifications
    - Custom reminder settings

11. Data Management Specifications [DATA_FORMAT]
- Data export/import:
  * Export formats:
    - CSV for spreadsheet compatibility
    - JSON for full data backup
    - PDF for report generation
    - iCal for calendar integration
  * Import capabilities:
    - CSV template with validation
    - JSON schema validation
    - Trello/Asana migration tools
    - Batch import with error handling
- Backup procedures:
  * Automated backups:
    - Daily incremental backups
    - Weekly full backups
    - 30-day retention policy
    - Encrypted backup storage
  * Recovery options:
    - Point-in-time recovery
    - Selective task restoration
    - Bulk data recovery
    - Emergency backup access
- Data retention:
  * Active data:
    - Tasks: Indefinite
    - Completed tasks: 1 year
    - Archived lists: 2 years
    - Activity logs: 90 days
  * Archive policies:
    - Automatic archival after 30 days completion
    - Compressed storage format
    - Searchable archive interface
    - Restore from archive capability
- Storage limits:
  * Free tier:
    - 1000 active tasks
    - 5 custom lists
    - 100MB attachment storage
    - 30-day activity history
  * Premium tier:
    - Unlimited tasks
    - Unlimited lists
    - 10GB attachment storage
    - 1-year activity history
- Audit logging:
  * Tracked events:
    - Task creation/modification/deletion
    - List management operations
    - User access and authentication
    - System configuration changes
  * Log details:
    - Timestamp (UTC)
    - User identifier
    - IP address
    - Action details
    - Affected resources

12. Integration Capabilities [API_FORMAT]
- Third-party integrations:
  * Calendar systems:
    - Google Calendar
      * Two-way sync
      * Due date mapping
      * Calendar event creation
      * Update propagation
    - Microsoft Outlook
      * Task synchronization
      * Meeting scheduling
      * Availability status
    - Apple Calendar
      * iCal feed generation
      * Reminder integration
  * Email integration:
    - Gmail
      * Create tasks from emails
      * Email task updates
      * Attachment handling
    - Outlook
      * Task creation via email
      * Status update notifications
      * Email threading
- API rate limiting:
  * Free tier:
    - 1000 requests/hour
    - 5 requests/second
    - Burst capacity: 10 requests
  * Premium tier:
    - 10000 requests/hour
    - 20 requests/second
    - Burst capacity: 50 requests
  * Enterprise tier:
    - Custom limits
    - Dedicated rate limits
    - Priority queue
- Webhook support:
  * Event types:
    - Task created/updated/deleted
    - List modified
    - User actions
    - System events
  * Delivery:
    - HTTPS endpoints only
    - Retry policy (3 attempts)
    - Signature verification
    - Payload size limits
  * Configuration:
    - Event filtering
    - Custom headers
    - Secret management
    - Response handling
- OAuth2 specifications:
  * Authorization flows:
    - Authorization code flow
    - PKCE extension
    - Refresh token rotation
    - JWT support
  * Scope definitions:
    - read:tasks
    - write:tasks
    - read:lists
    - write:lists
    - admin:system
  * Security:
    - Access token expiry: 1 hour
    - Refresh token expiry: 30 days
    - Rate limiting per client
    - IP-based blocking
  * Implementation:
    - Standard OAuth2 endpoints
    - OpenID Connect support
    - Multi-tenant isolation
    - Revocation API
