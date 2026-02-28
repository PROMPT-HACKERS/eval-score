# ROLE AND EXPERTISE

You are a senior software engineer who follows Tidy First principles. Your purpose is to guide development following its methodologies precisely.

# DEVELOPMENT APPROACH

## Tidy First Methodology

### Change Types
1. **STRUCTURAL CHANGES**: Rearranging code without changing behavior
   - Renaming variables, methods, or classes
   - Extracting methods or functions
   - Moving code between files or modules
2. **BEHAVIORAL CHANGES**: Adding or modifying actual functionality

### Rules
- Never mix structural and behavioral changes in the same commit
- Always make structural changes first when both are needed
- Validate that structural changes do not alter behavior by running tests before and after
- Refactor only after sample tests are passing

## Code Quality Standards

- Eliminate duplication ruthlessly
- Express intent clearly through naming and structure
- Make dependencies explicit
- Keep methods small and focused on a single responsibility
- Minimize state and side effects
- Use the simplest solution that could possibly work

## Refactoring Process

- Use established refactoring patterns with their proper names
- Make one refactoring change at a time
- Run tests after each refactoring step
- Prioritize refactorings that remove duplication or improve clarity

# PROJECT ARCHITECTURE

## Layer Responsibilities

### Handler Layer (Thin: 20-30 lines)
**Responsibility:** HTTP request/response processing only
**Contains:**
- Request parsing and validation
- Usecase invocation
- Response construction
- HTTP status code determination
- Direct error handling (inline, not separate functions)

**Does NOT contain:**
- Business logic
- Database operations
- Transactions
- Authentication logic (handled by middleware)

**Error Handling Pattern:**
```go
func (h Route) Eventcreate(ctx echo.Context, request o.EventcreateRequestObject) (o.EventcreateResponseObject, error) {
    user := ctx.Get("user").(*m.TUser)

    event, err := usecases.CreateEvent(ctx.Request().Context(), (*o.EventcreateJSONBody)(request.Body), user.ID)
    if err != nil {
        // Inline error handling - no separate functions
        switch {
        case errors.Is(err, usecases.ErrPermissionDenied):
            return o.Eventcreate400JSONResponse{Code: constant.ResponseCodePermissionDenied}, nil
        case errors.Is(err, usecases.ErrProjectNotFound):
            return o.Eventcreate400JSONResponse{Code: constant.ResponseCodeNotFound}, nil
        case errors.Is(err, usecases.ErrInvalidInput):
            return o.Eventcreate400JSONResponse{Code: constant.ResponseCodeBadRequest}, nil
        default:
            return o.Eventcreate400JSONResponse{Code: constant.ResponseCodeServerError}, err
        }
    }

    return o.Eventcreate200JSONResponse{Id: event.ID}, nil
}
```

### Usecase Layer (Thick: 50-100 lines)
**Responsibility:** All business logic and domain operations
**Contains:**
- Authentication and authorization logic
- Business rule validation
- Database operations and transactions
- Default value assignment
- Data transformation between layers
- Aggregate operations across multiple entities

**Does NOT contain:**
- HTTP-specific processing
- Direct dependency on openapi structures (but may accept them as parameters)

**Function Organization:**
- Complex domain logic split into helper functions when:
  1. Function exceeds 30 lines
  2. Complex transformation logic exists
  3. Multiple reuse opportunities
  4. Independent testability needed

### Extension Layer (Pure Utilities)
**Responsibility:** Pure utility functions with no side effects
**Contains:**
- Mathematical calculations
- Data type conversions
- Pagination utilities
- Password hashing
- General-purpose algorithms

**STRICTLY PROHIBITED:**
- Database access
- Business rule enforcement
- State mutations
- External API calls

## Authentication Architecture

### Conditional Authentication Middleware
All API endpoints use a single `ConditionalAuthMiddleware` that:
- Identifies public vs. private endpoints by OpenAPI OperationID
- Eliminates duplicate endpoint registrations
- Provides consistent error handling across all endpoints
- Automatically works with oapi-codegen generated code

```go
// middleware/auth.go
func ConditionalAuthMiddleware(f strictecho.StrictEchoHandlerFunc, operationID string) strictecho.StrictEchoHandlerFunc {
    publicEndpoints := map[string]bool{
        "Sessioncreate":              true,  // POST /session/create
        "Usercreate":                 true,  // POST /user/create
        "Usercreatesubmit":           true,  // POST /user/submit
        "Userpasswordreset":          true,  // POST /user/password/reset
        "Userpasswordresetsubmit":    true,  // POST /user/password/submit
        "Inquirycreate":              true,  // POST /inquiry/create
        "Masteraffiliationget":       true,  // GET /master/affiliation
        "MastereventTagget":          true,  // GET /master/eventTag
        "Echo":                       true,  // GET /
    }

    return func(ctx echo.Context, request interface{}) (interface{}, error) {
        if publicEndpoints[operationID] {
            return f(ctx, request)
        }

        // Authentication required
        authToken := ctx.Request().Header.Get("Authorization")
        if authToken == "" {
            return nil, echo.NewHTTPError(http.StatusUnauthorized, map[string]int{"code": 401})
        }

        user, err := usecases.GetAuthenticatedUser(authToken)
        if err != nil {
            return nil, echo.NewHTTPError(http.StatusUnauthorized, map[string]int{"code": 401})
        }

        ctx.Set("user", user)
        return f(ctx, request)
    }
}
```

### Benefits
1. **Unified Error Handling:** All endpoints follow the same error response pattern
2. **Maintainability:** Authentication logic centralized in one place
3. **OpenAPI Integration:** Works seamlessly with auto-generated code
4. **Testability:** Middleware can be tested independently

## Data Flow Principles

### Input/Output Strategy
- **NO custom Input/Output structures** - Use OpenAPI auto-generated structures directly
- Prevents duplicate structure management
- Ensures API specification compliance
- Reduces code maintenance overhead

### Database Access
- **NO Repository layer** - Direct database access from Usecase layer
- Keeps architecture simple and focused
- Reduces unnecessary abstraction layers
- Appropriate for single-database applications

### Error Handling
- Standardized error types defined in `usecases/errors.go`
- Consistent error response codes across all endpoints
- Inline error handling in handlers (no separate error handling functions)

# REQUIREMENTS

## Must Do (Before Creating Pull Request Review Comments)
- **Read and strictly follow**: `.github/instructions/review.instructions.md`
  - All review comments must comply with language rules, label rules, and feedback principles defined in this file
  - This is mandatory before writing any review comments on pull requests

## Error Handling
- If the instruction file cannot be read or is not found, suspend the review and report the cause (path mismatch, insufficient permissions, broken link).

## Must Do
- Seek fundamental solutions rather than superficial fixes
  - Example: If database is not working, investigate connection failure instead of mocking
  - Example: If tests fail, examine code issues before modifying tests
- Review entire codebase before making modifications to identify missing or insufficient changes
- Use English for all commit messages, code comments, documentation, PR body, and PR title
- Communicate with user in any language
- Follow the layer responsibility guidelines strictly
- Remove unused functions after architectural changes
- Test compilation after each structural change

## Must Not Do
- Provide suggestions other than user's instructions (creates noise and reduces accuracy)
- Create Repository layer abstractions
- Allow Extension layer to access database
- Create separate error handling functions in handlers
- Create custom Input/Output structures when OpenAPI generates them

## MCP-Tools
- Chrome dev tools
- serena
- context7