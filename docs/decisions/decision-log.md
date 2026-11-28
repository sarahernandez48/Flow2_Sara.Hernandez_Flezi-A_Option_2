Scope: High = original specification requirements and agreed core implementation decisions. Low = enhancements deferred until core completion.

| ID | Decision | Choice | Why | Priority |
|---|---|---|---|---|
| D01 | Backend framework | .NET Minimal API | Lightweight backend for the required CRUD API. | High |
| D02 | Data access | Entity Framework Core | Agreed persistence, querying, and migrations approach. | High |
| D03 | Database | SQLite | Relational storage with simple local setup. | High |
| D04 | Frontend | React + Vite + TypeScript | Agreed reactive, typed frontend stack. | High |
| D05 | Backend testing framework | xUnit | Agreed framework for automated backend tests. | High |
| D06 | Backend architecture | Lightweight feature/vertical organization | Groups functionality by operation without extra layers. | High |
| D07 | Authentication | JWT Bearer with 2–3 predefined development users | Provides authentication without an external identity provider. | High |
| D08 | Authentication failure | `401 Unauthorized` for missing, invalid, or expired JWT | Protected endpoints require a valid authenticated user. | High |
| D09 | Ownership identity | Derive `OwnerId` exclusively from JWT claims | Prevents the client from selecting another owner. | High |
| D10 | Cross-owner access | `404 Not Found` | Indistinguishable from a nonexistent task; `200 {}` and `403` rejected. | High |
| D11 | Ownership enforcement | Filter task operations by both task `Id` and authenticated `OwnerId` | Prevents reading or modifying another user's task. | High |
| D12 | Task identifier | UUID | Agreed primary-key type. | High |
| D13 | Core task model | `Id`, `OwnerId`, `Title`, `Description`, `Status`, `CreatedAtUtc`, `UpdatedAtUtc` | Meets the original model without deferred `TaskDate`. | High |
| D14 | Title validation | Required; trim; reject empty/whitespace; max 200 characters | Invalid title returns `400 Bad Request`. | High |
| D15 | Description validation | Optional; max 2,000 characters | Enforces the agreed input boundary. | High |
| D16 | Task status | `open` or `done`; default `open` | Matches the specified two-state model. | High |
| D17 | Status database constraint | `CHECK` allowing only `open` and `done` | Prevents invalid persisted values. | High |
| D18 | Timestamps | Automatically maintain creation and update timestamps | Meets the specified model. | High |
| D19 | Core ordering | `CreatedAtUtc DESC, Id DESC` | Newest first with deterministic tie-breaking; no `TaskDate` dependency. | High |
| D20 | Create endpoint | `POST /tasks` → `201 Created` with resource | Matches the specified create contract. | High |
| D21 | Create validation error | `400 Bad Request` | Rejects invalid required fields. | High |
| D22 | List endpoint | `GET /tasks` → `200 OK` | Returns only authenticated user's tasks. | High |
| D23 | Empty list | `200 OK` with empty collection | A successful query may have zero results. | High |
| D24 | Status filter | `GET /tasks?status=open` or `?status=done` | Implements required optional filtering. | High |
| D25 | Invalid status filter | `400 Bad Request` | Do not silently ignore invalid filter values. | High |
| D26 | GetById endpoint | **No** `GET /tasks/{id}` | Not required by the original contract; reduces scope. | High |
| D27 | Update endpoint | `PUT /tasks/{id}` → `200 OK` | Matches the specified update contract. | High |
| D28 | PUT semantics | Send complete desired editable state: `title`, `description`, `status` | Repeated identical requests must not toggle the state. | High |
| D29 | Update validation error | `400 Bad Request` | Rejects invalid editable fields. | High |
| D30 | Missing or cross-owner update | `404 Not Found` | Same response for absent and inaccessible resources. | High |
| D31 | Delete endpoint | `DELETE /tasks/{id}` → `204 No Content` | Permanently deletes an owned task. | High |
| D32 | Missing or cross-owner delete | `404 Not Found` | Includes already-deleted tasks. | High |
| D33 | Unexpected server error | `500 Internal Server Error` on every endpoint via centralized handler | Consistent failure handling without exposing internals. | High |
| D34 | Error response | ProblemDetails-style structured errors | Predictable client-facing error format. | High |
| D35 | API documentation | OpenAPI/Swagger | Documents models and response codes. | High |
| D36 | Validation library | **FluentValidation** | Separates validation rules from endpoint logic; invalid requests return `400`. | High |
| D37 | Frontend server-state management | **TanStack Query** | Handles caching, invalidation, mutations, and optimistic updates. | High |
| D38 | Task UI | Add, list, inline edit, toggle status, delete | Covers required user interactions. | High |
| D39 | Completed-task presentation | Visually distinguish `done` tasks, e.g. strikethrough | Required frontend behavior. | High |
| D40 | Empty-state presentation | Show message when no tasks exist | Required frontend behavior. | High |
| D41 | Rapid-click handling | Per-task pending-request lock | Prevents duplicate concurrent requests without locking the whole list. | High |
| D42 | Optimistic UI | Optimistic changes with rollback for failed operations, except reconciled deletion | Provides immediate feedback while preserving confirmed state. | High |
| D43 | Failed delete rollback | Restore optimistically removed task for failures other than `404` | A non-`404` failure does not confirm desired deletion. | High |
| D44 | Already-deleted reconciliation | On DELETE `404`, keep item removed and show informational toast | The UI's desired absent state is already achieved. | High |
| D45 | Failed status update rollback | Restore previous status | Do not retain an unconfirmed status change. | High |
| D46 | Failed edit rollback | Restore previous task values | Do not retain an unconfirmed edit. | High |
| D47 | Write feedback | Success, error, or informational toast as appropriate | Required write-action feedback. | High |
| D48 | Test scenario 1 | xUnit isolated test: whitespace-only title fails FluentValidation; endpoint mapping yields `400` | Covers invalid-title scenario. | High |
| D49 | Test scenario 2 | xUnit isolated test using SQLite in-memory: user B updates/deletes user A's task → `404`, task unchanged | Covers cross-owner denial with the real ownership query executing. | High |
| D50 | Test scenario 3 | xUnit isolated test: explicit PUT `open → done → open`; identical retries preserve state | Covers status changes and idempotency. | High |
| D51 | Test scenario 4 | xUnit isolated test: delete nonexistent ID → `404` | Covers missing-resource deletion. | High |
| D52 | Test scenario 5 | xUnit isolated test using SQLite in-memory with mixed-owner/mixed-status data: `?status=open` returns only the current user's open tasks | Covers status filtering and ownership with the real query executing. | High |
| D53 | Test approach | xUnit isolated unit tests: operation handlers tested directly (no HTTP pipeline); persistence via SQLite in-memory, not mocked EF Core | Mocked EF would bypass the ownership query filter (D11), so tests could pass with a broken filter. Integration tests not selected. | High |
| D54 | README | How to run app and tests; completed estimation | Explicit project deliverable. | High |
| D55 | Task business date | Defer explicit `TaskDate` | Not required for core CRUD. | Low |
| D56 | Task uniqueness | Later: `UNIQUE(OwnerId, NormalizedTitle, TaskDate)` | Prevents same user's duplicate title on same business date. | Low |
| D57 | Uniqueness normalization | Later: trim and compare titles case-insensitively | Prevents casing/whitespace variants bypassing uniqueness. | Low |
| D58 | Duplicate POST | Later: `409 Conflict` for same owner/normalized title/date | Only applies after uniqueness is implemented. | Low |
| D59 | Duplicate PUT | Later: `409 Conflict` when editing violates unique owner/title/date | Only applies after uniqueness is implemented. | Low |
| D60 | Large task lists | Later: traditional offset pagination with `page` and `pageSize` | Simpler than cursor pagination. | Low |
| D61 | Pagination bounds | Later: default `page=1`, `pageSize=20`; maximum `pageSize=100` | Bounds payload size. | Low |
| D62 | Invalid pagination | Later: `400 Bad Request` for invalid page/pageSize | Explicit invalid-input behavior. | Low |
| D63 | Pagination response | Later: `items`, `page`, `pageSize`, `totalCount`, `totalPages` | Supports conventional page controls. | Low |
| D64 | Pagination ordering | Later: use core ordering `CreatedAtUtc DESC, Id DESC` | Stable order for offset pages. | Low |
| D65 | Cursor pagination | Rejected; not planned | Adds complexity beyond project scope. | Low |
| D66 | Cross-tab synchronization | Later: browser `BroadcastChannel` | Synchronizes same-browser tabs without server push. | Low |
| D67 | Server real-time sync | SignalR excluded from current scope | Multi-device server-push synchronization not required. | Low |
| D68 | JSON field naming | API JSON uses spec names `createdAt` / `updatedAt`; C# properties remain `CreatedAtUtc` / `UpdatedAtUtc` | Keeps the external contract identical to the spec. | High |
| D69 | Project structure | See section D69 below; code internals confirmed per Codex task | Tree is a plan, not a binding contract for internals. | High |

TaskToDoManagement/
│
├── AGENTS.md                              # AI coding instructions, conventions, and project boundaries
├── README.md                              # Setup, execution, testing.
├── .gitignore                             # Excludes build artifacts, secrets, and local databases
├── .editorconfig                          # Shared formatting conventions
├── TaskToDoManagement.sln                 # .NET solution
│
├── backend/
│   ├── src/
│   │   └── TaskToDo.Api/
│   │       ├── TaskToDo.Api.csproj         # Backend project dependencies
│   │       ├── Program.cs                  # Application startup and endpoint registration
│   │       ├── appsettings.json            # Non-sensitive application settings
│   │       ├── appsettings.Development.json # Local development configuration
│   │       │
│   │       ├── Authentication/
│   │       │   ├── JwtOptions.cs           # JWT configuration model
│   │       │   ├── JwtTokenService.cs      # Development JWT generation
│   │       │   ├── DevelopmentUsers.cs     # Predefined development user definitions
│   │       │   ├── CurrentUser.cs          # Authenticated user identity from JWT claims
│   │       │   ├── LoginEndpoint.cs        # Development-only login endpoint
│   │       │   └── LoginContracts.cs       # Login request and response DTOs
│   │       │
│   │       ├── Domain/
│   │       │   ├── TaskItem.cs             # Task entity and UTC timestamps
│   │       │   └── TaskStatus.cs           # Open and done status values
│   │       │
│   │       ├── Features/
│   │       │   └── Tasks/
│   │       │       ├── Create/
│   │       │       │   ├── CreateTaskEndpoint.cs   # HTTP POST and response mapping
│   │       │       │   ├── CreateTaskHandler.cs    # Testable task creation logic
│   │       │       │   ├── CreateTaskRequest.cs    # Create request DTO
│   │       │       │   └── CreateTaskValidator.cs  # FluentValidation title/description rules
│   │       │       │
│   │       │       ├── GetList/
│   │       │       │   ├── GetTasksEndpoint.cs     # HTTP GET and response mapping
│   │       │       │   ├── GetTasksHandler.cs      # Testable ownership, filtering, and ordering
│   │       │       │   ├── GetTasksQuery.cs        # Optional status query
│   │       │       │   └── GetTasksValidator.cs    # FluentValidation status filter rules
│   │       │       │
│   │       │       ├── Update/
│   │       │       │   ├── UpdateTaskEndpoint.cs   # HTTP PUT and response mapping
│   │       │       │   ├── UpdateTaskHandler.cs    # Testable ownership and idempotent updates
│   │       │       │   ├── UpdateTaskRequest.cs    # Complete editable task state
│   │       │       │   └── UpdateTaskValidator.cs  # FluentValidation update rules
│   │       │       │
│   │       │       ├── Delete/
│   │       │       │   ├── DeleteTaskEndpoint.cs   # HTTP DELETE and response mapping
│   │       │       │   └── DeleteTaskHandler.cs    # Testable ownership and deletion logic
│   │       │       │
│   │       │       └── Shared/
│   │       │           ├── TaskResponse.cs        # JSON DTO with createdAt/updatedAt
│   │       │           └── TaskEndpoints.cs        # Registers the four core task endpoints
│   │       │
│   │       ├── Infrastructure/
│   │       │   └── Persistence/
│   │       │       ├── AppDbContext.cs             # EF Core SQLite context
│   │       │       ├── TaskItemConfiguration.cs    # Indexes, lengths, and CHECK constraint
│   │       │       └── Migrations/                 # EF Core database migrations
│   │       │
│   │       ├── Common/
│   │       │   ├── GlobalExceptionHandler.cs      # Centralized 500/ProblemDetails handling
│   │       │   ├── ValidationExtensions.cs        # FluentValidation endpoint integration
│   │       │   └── ServiceRegistration.cs         # Authentication, persistence, and DI setup
│   │       │
│   │       └── Properties/
│   │           └── launchSettings.json            # Local API launch profiles
│   │
│   └── tests/
│       └── TaskToDo.Tests/
│           ├── TaskToDo.Tests.csproj               # xUnit and SQLite in-memory dependencies
│           │
│           ├── Fixtures/
│           │   ├── SqliteTestDatabase.cs           # Shared SQLite in-memory setup and lifecycle
│           │   └── TestDataFactory.cs              # Users and task test data builders
│           │
│           ├── Validation/
│           │   ├── CreateTaskValidatorTests.cs     # Empty/whitespace title and length validation
│           │   ├── UpdateTaskValidatorTests.cs     # Update request validation
│           │   └── GetTasksValidatorTests.cs       # Invalid status filtering
│           │
│           └── Tasks/
│               ├── CreateTaskHandlerTests.cs       # Creation and default open status
│               ├── GetTasksHandlerTests.cs         # Ownership, filtering, and core ordering
│               ├── UpdateTaskHandlerTests.cs       # Ownership, status changes, and idempotency
│               └── DeleteTaskHandlerTests.cs       # Ownership, missing IDs, and deletion
│
├── frontend/
│   ├── package.json                                # React, Vite, TypeScript, and TanStack Query
│   ├── package-lock.json                           # Locked npm dependencies
│   ├── index.html                                  # Application HTML entry
│   ├── vite.config.ts                              # Vite development/build configuration
│   ├── tsconfig.json                               # TypeScript configuration
│   ├── tsconfig.app.json                           # Frontend TypeScript settings
│   ├── .env.example                                # Example API base URL configuration
│   │
│   ├── public/                                     # Static assets
│   │
│   └── src/
│       ├── main.tsx                                # React application bootstrap
│       ├── App.tsx                                 # Application composition and routing
│       │
│       ├── auth/
│       │   ├── authApi.ts                          # Development login API client
│       │   ├── AuthProvider.tsx                    # JWT and authenticated user state
│       │   ├── LoginForm.tsx                       # Development user login UI
│       │   └── useAuth.ts                          # Authentication access hook
│       │
│       ├── features/
│       │   └── tasks/
│       │       ├── api/
│       │       │   └── tasksApi.ts                  # Core CRUD HTTP requests
│       │       │
│       │       ├── components/
│       │       │   ├── TaskList.tsx                 # Renders current user's tasks
│       │       │   ├── TaskItem.tsx                 # Task display and action controls
│       │       │   ├── TaskForm.tsx                 # Task creation form
│       │       │   ├── TaskInlineEdit.tsx           # Inline title/description editing
│       │       │   ├── TaskFilters.tsx              # All/Open/Done filtering controls
│       │       │   └── TaskEmptyState.tsx           # Empty-list presentation
│       │       │
│       │       ├── hooks/
│       │       │   ├── useTasks.ts                  # TanStack Query task retrieval
│       │       │   └── useTaskMutations.ts          # Mutations, locks, rollback, reconciliation
│       │       │
│       │       ├── types/
│       │       │   └── task.types.ts                # Task DTOs with createdAt/updatedAt
│       │       │
│       │       └── TasksPage.tsx                    # Main task-management screen
│       │
│       ├── shared/
│       │   ├── api/
│       │   │   ├── httpClient.ts                    # JWT-aware HTTP client
│       │   │   └── apiErrors.ts                     # HTTP and ProblemDetails error mapping
│       │   │
│       │   ├── components/
│       │   │   ├── Layout.tsx                       # Shared page layout
│       │   │   └── ToastProvider.tsx                # Success, error, and informational toasts
│       │   │
│       │   └── queryClient.ts                      # TanStack Query client configuration
│       │
│       └── styles/
│           ├── global.css                          # Global UI styles
│           └── tasks.css                           # Task list, edit, and completed-state styles
│
├── docs/
│   ├── decisions/
│   │   ├── api-contract.md                         # Agreed endpoints, DTOs, and status codes
│   │   └── decision-log.md                         # Final High/Low priority decision record
│   │
│   ├── requirements/
│   │   └── TaskTo-Do-CRUD-Management.md             # Original source specification
│   │
│   ├── adr/
│   │   └── README.md                               # ADR index and future decision record convention
│   │
│   ├── test-plan.md                                # Test approach, five scenarios, and edge cases
│   └── traceability-matrix.md                      # Requirement-to-handler-to-test mapping