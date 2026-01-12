# Backend API Endpoints Documentation

This document provides a comprehensive catalog of all API endpoints implemented in the CostPie backend application, following AWS documentation standards and organizational patterns.

## API Architecture Overview

### Base Configuration

The backend exposes a RESTful API with the following standards:

```
Base URL: http://localhost:3001 (development) | https://costpie-backend.onrender.com (production)
Authentication: Bearer token (JWT from Supabase)
Authorization: Organization-scoped access via AuthGuard
API Documentation: Swagger UI available at /api/docs
```

### Request Standards

1. **Authentication**: All endpoints require Bearer token authentication via AuthGuard
2. **Organization Context**: Organization ID passed via URL parameter for scoped operations
3. **Content Type**: `application/json` for all request/response bodies
4. **Versioning**: Version 1 implied (some endpoints use explicit v1 prefix)
5. **Error Handling**: Standardized HTTP status codes with descriptive error messages


## Backend API Architecture Workflow

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│    HTTP Client  │───▶│   NestJS Router  │───▶│   Controllers   │
│   (Frontend)    │    │   (app.module)   │    │   (@Controller) │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       ▼
         │                       │              ┌─────────────────┐
         │                       │              │   Auth Guard    │
         │                       │              │ (JWT Validation)│
         │                       │              └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Response      │◀───│   Swagger/Docs   │    │    Services     │
│   Interceptor   │    │   (API Schema)   │    │ (Business Logic)│
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                                              │
         │                                              ▼
         │                              ┌─────────────────────────┐
         │                              │     Data Layer          │
         │                              │  ┌─────────────────┐   │
         │                              │  │  Prisma ORM     │   │
         │                              │  │  (Database)     │   │
         │                              │  └─────────────────┘   │
         │                              │  ┌─────────────────┐   │
         │                              │  │  AWS Services   │   │
         │                              │  │  (External API) │   │
         │                              │  └─────────────────┘   │
         │                              │  ┌─────────────────┐   │
         │                              │  │  Vault Service  │   │
         │                              │  │  (Encryption)   │   │
         │                              │  └─────────────────┘   │
         │                              └─────────────────────────┘
         │                                              │
         └──────────────────────────────────────────────┘

         ┌─────────────────────────────────────────────────────┐
         │             Background Processes                     │
         │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
         │  │   Sync      │  │ CloudTrail  │  │ Scheduled   │ │
         │  │   Jobs      │  │   Events    │  │ Resources   │ │
         │  └─────────────┘  └─────────────┘  └─────────────┘ │
         └─────────────────────────────────────────────────────┘
```

## API Endpoint Structure

### Organizations Module

**Base Path**: `/organizations`
**Guard**: AuthGuard (JWT Required)
**Controller**: `OrganizationsController`

| Method | Endpoint                   | Description              | Request Body          | Response               |
| ------ | -------------------------- | ------------------------ | --------------------- | ---------------------- |
| POST   | `/organizations`         | Create new organization  | CreateOrganizationDto | Organization object    |
| GET    | `/organizations/mine`    | Get user's organizations | -                     | Array of organizations |
| PUT    | `/organizations/{orgId}` | Update organization      | UpdateOrganizationDto | Updated organization   |

**Request/Response Examples:**

```typescript
// POST /organizations
{
  "name": "My Organization",
  "description": "Organization description"
}

// Response
{
  "id": "uuid",
  "name": "My Organization",
  "description": "Organization description",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z"
}
```

### Cloud Accounts Module

**Base Path**: Mixed paths with organization and cloud-account prefixes
**Guard**: AuthGuard (JWT Required)
**Controller**: `CloudAccountsController`

| Method | Endpoint                                               | Description                   | Request Body          | Query Parameters                                     |
| ------ | ------------------------------------------------------ | ----------------------------- | --------------------- | ---------------------------------------------------- |
| POST   | `/organizations/{orgId}/cloud-accounts`              | Add cloud account             | CreateCloudAccountDto | -                                                    |
| GET    | `/organizations/{orgId}/cloud-accounts`              | List cloud accounts           | -                     | -                                                    |
| GET    | `/cloud-accounts/{accountId}`                        | Get cloud account details     | -                     | -                                                    |
| PUT    | `/cloud-accounts/{accountId}`                        | Update cloud account          | UpdateCloudAccountDto | -                                                    |
| DELETE | `/cloud-accounts/{accountId}`                        | Delete cloud account          | -                     | -                                                    |
| POST   | `/cloud-accounts/{accountId}/sync`                   | Trigger resource sync (async) | -                     | -                                                    |
| GET    | `/cloud-accounts/{accountId}/aws-costs`              | Get AWS cost data             | -                     | startDate, endDate, granularity, groupBy, groupByKey |
| GET    | `/cloud-accounts/{accountId}/network-topology`       | Get network topology          | -                     | -                                                    |
| GET    | `/cloud-accounts/{accountId}/resource-relationships` | Get resource relationships    | -                     | -                                                    |

**Sync Job Response Example:**

```json
{
  "success": true,
  "message": "Sync job created and queued for processing",
  "jobId": "uuid",
  "status": "pending",
  "progress": 0,
  "progressMessage": "Job queued"
}
```

### Resources Module

**Base Path**: Mixed paths with cloud-account and resource prefixes
**Guard**: AuthGuard (JWT Required)
**Controller**: `ResourcesController`

| Method | Endpoint                                        | Description                 | Request Body | Query Parameters                                          |
| ------ | ----------------------------------------------- | --------------------------- | ------------ | --------------------------------------------------------- |
| GET    | `/cloud-accounts/{accountId}/resources`       | List resources with filters | -            | type, region, status, tags, created_after, created_before |
| GET    | `/cloud-accounts/{accountId}/resources/count` | Get resource count          | -            | ResourceFilterDto parameters                              |
| GET    | `/cloud-accounts/{accountId}/resources/stats` | Get resource statistics     | -            | ResourceFilterDto parameters                              |
| GET    | `/resources/{resourceId}`                     | Get resource details        | -            | -                                                         |
| POST   | `/resources/{resourceId}/team/{teamId}`       | Assign resource to team     | -            | -                                                         |
| DELETE | `/resources/{resourceId}/team`                | Remove resource from team   | -            | -                                                         |

**Resource Filter Parameters:**

- `type`: Resource type (e.g., ec2_instance, rds_instance)
- `region`: AWS region
- `status`: Resource status
- `tags`: Tag-based filtering
- `created_after`: ISO date string
- `created_before`: ISO date string

### Costs & Analytics Module

**Base Path**: Organization-scoped paths
**Guard**: AuthGuard (JWT Required)
**Controller**: `CostsController`

| Method | Endpoint                                            | Description                      | Request Body | Query Parameters                      |
| ------ | --------------------------------------------------- | -------------------------------- | ------------ | ------------------------------------- |
| GET    | `/organizations/{orgId}/cost-trend`               | Get cost trends                  | -            | CostTrendParamsDto                    |
| GET    | `/organizations/{orgId}/cost-trend-with-forecast` | Cost with forecast               | -            | CostTrendParamsDto + forecast options |
| GET    | `/organizations/{orgId}/cloud-health`             | Get cloud health metrics         | -            | -                                     |
| GET    | `/organizations/{orgId}/optimization-lab`         | Get optimization recommendations | -            | -                                     |
| POST   | `/organizations/{orgId}/sync-optimization-data`   | Sync optimization data           | -            | -                                     |
| GET    | `/organizations/{orgId}/savings`                  | Get savings data                 | -            | startDate, endDate                    |
| GET    | `/organizations/{orgId}/resource-counts`          | Get resource counts              | -            | startDate, endDate                    |
| GET    | `/organizations/{orgId}/utilization-metrics`      | Get utilization metrics          | -            | startDate, endDate                    |
| POST   | `/organizations/{orgId}/sync-cost-data`           | Manual cost data sync            | -            | startDate, endDate, granularity       |
| POST   | `/organizations/{orgId}/full-historical-sync`     | Full 12-month sync               | -            | -                                     |
| GET    | `/organizations/{orgId}/sync-status`              | Get sync status                  | -            | -                                     |

**Forecast Options:**

- `includeForecast`: boolean
- `forecastDays`: number (default: 30)
- `forecastMethod`: 'aws-api' | 'trend-based' | 'resource-based'
- `confidenceLevel`: number (default: 80)

### Tags Module

**Base Path**: Organization and cloud-account scoped paths
**Guard**: AuthGuard (JWT Required)
**Controller**: `TagsController`

| Method | Endpoint                                                | Description               | Request Body | Query Parameters                            |
| ------ | ------------------------------------------------------- | ------------------------- | ------------ | ------------------------------------------- |
| GET    | `/organizations/{orgId}/tags`                         | List organization tags    | -            | key, value, cloud_account_id, resource_type |
| GET    | `/cloud-accounts/{accountId}/tags`                    | List account tags         | -            | key, value                                  |
| POST   | `/organizations/{orgId}/tags`                         | Create and apply tag      | CreateTagDto | -                                           |
| GET    | `/organizations/{orgId}/tags/{key}/{value}`           | Get tag details           | -            | -                                           |
| POST   | `/organizations/{orgId}/tags/{key}/{value}/apply`     | Apply tag to resources    | ApplyTagDto  | -                                           |
| POST   | `/organizations/{orgId}/tags/{key}/{value}/remove`    | Remove tag from resources | ApplyTagDto  | -                                           |
| DELETE | `/organizations/{orgId}/tags/{key}/{value}`           | Delete tag completely     | -            | -                                           |
| POST   | `/organizations/{orgId}/tags/batch`                   | Batch tag operations      | BatchTagDto  | -                                           |
| GET    | `/organizations/{orgId}/tags/{key}/{value}/resources` | Get resources by tag      | -            | -                                           |
| GET    | `/organizations/{orgId}/tags/untagged-resources`      | Get untagged resources    | -            | -                                           |
| GET    | `/organizations/{orgId}/tags/summary`                 | Get tags summary          | -            | -                                           |

**Tag Operation Examples:**

```typescript
// Create Tag DTO
{
  "key": "Environment",
  "value": "Production",
  "resource_ids": ["resource-uuid-1", "resource-uuid-2"]
}

// Batch Tag DTO
{
  "operations": [
    {
      "action": "apply",
      "key": "Environment",
      "value": "Production",
      "resource_ids": ["resource-1", "resource-2"]
    }
  ]
}
```

### Users Module

**Base Path**: Users and organization-scoped paths
**Guard**: AuthGuard (JWT Required)
**Controller**: `UsersController`

| Method | Endpoint                              | Description             | Request Body  | Query Parameters                  |
| ------ | ------------------------------------- | ----------------------- | ------------- | --------------------------------- |
| GET    | `/users/me`                         | Get current user        | -             | -                                 |
| PATCH  | `/users/me`                         | Update current user     | UpdateUserDto | -                                 |
| GET    | `/organizations/{orgId}/users`      | List organization users | -             | page, limit, search, role, status |
| POST   | `/organizations/{orgId}/users`      | Create user             | CreateUserDto | -                                 |
| GET    | `/users/{userId}`                   | Get user by ID          | -             | -                                 |
| PATCH  | `/users/{userId}`                   | Update user             | UpdateUserDto | -                                 |
| DELETE | `/users/{userId}`                   | Delete user             | -             | -                                 |
| POST   | `/users/{userId}/resend-invitation` | Resend invitation       | -             | -                                 |
| GET    | `/users/{userId}/teams`             | Get user teams          | -             | -                                 |

### Teams Module

**Base Path**: Organization and teams scoped paths
**Guard**: AuthGuard (JWT Required)
**Controller**: `TeamsController`

| Method | Endpoint                             | Description             | Request Body        | Query Parameters |
| ------ | ------------------------------------ | ----------------------- | ------------------- | ---------------- |
| GET    | `/organizations/{orgId}/teams`     | List organization teams | -                   | -                |
| POST   | `/organizations/{orgId}/teams`     | Create team             | CreateTeamDto       | -                |
| GET    | `/teams/{id}`                      | Get team by ID          | -                   | -                |
| PATCH  | `/teams/{id}`                      | Update team             | UpdateTeamDto       | -                |
| DELETE | `/teams/{id}`                      | Delete team             | -                   | -                |
| GET    | `/teams/{id}/members`              | Get team members        | -                   | -                |
| POST   | `/teams/{id}/members`              | Add team member         | AddTeamMemberDto    | -                |
| PATCH  | `/teams/{teamId}/members/{userId}` | Update team member      | UpdateTeamMemberDto | -                |
| DELETE | `/teams/{teamId}/members/{userId}` | Remove team member      | -                   | -                |

### AWS Integration Module

**Base Path**: `/aws` (versioned: v1)
**Guard**: AuthGuard (JWT Required)
**Controller**: `AwsController`

| Method | Endpoint                                               | Description                      | Request Body       | Query Parameters                         |
| ------ | ------------------------------------------------------ | -------------------------------- | ------------------ | ---------------------------------------- |
| POST   | `/aws/validate-credentials`                          | Validate AWS credentials         | Credentials object | -                                        |
| POST   | `/aws/cloud-accounts/{accountId}/discover-resources` | Discover AWS resources           | -                  | -                                        |
| GET    | `/aws/cloud-accounts/{accountId}/costs`              | Get AWS cost data                | -                  | startDate, endDate, granularity, groupBy |
| POST   | `/aws/cloud-accounts/{accountId}/recommendations`    | Get optimization recommendations | -                  | -                                        |
| POST   | `/aws/resources/{resourceId}/control`                | Control resource (start/stop)    | {action: "start"   | "stop"}                                  |

**AWS Credentials Format:**

```typescript
{
  "auth_type": "api_key" | "role_arn",
  "credentials": {
    "accessKeyId": "string",
    "secretAccessKey": "string"
  },
  "region": "us-east-1"
}
```

### Audit Logs Module

**Base Path**: `/organizations/{orgId}/audit-logs` (versioned: v1)
**Guard**: AuthGuard (JWT Required)
**Controller**: `AuditLogsController`

| Method | Endpoint                                   | Description            | Request Body      | Query Parameters  |
| ------ | ------------------------------------------ | ---------------------- | ----------------- | ----------------- |
| POST   | `/organizations/{orgId}/audit-logs`      | Create audit log entry | CreateAuditLogDto | -                 |
| GET    | `/organizations/{orgId}/audit-logs`      | List audit logs        | -                 | QueryAuditLogsDto |
| GET    | `/organizations/{orgId}/audit-logs/{id}` | Get specific audit log | -                 | -                 |

### Reservations Module

**Base Path**: Organization-scoped paths
**Guard**: AuthGuard (JWT Required)
**Controller**: `ReservationsController`

| Method | Endpoint                                        | Description                       | Request Body | Query Parameters |
| ------ | ----------------------------------------------- | --------------------------------- | ------------ | ---------------- |
| GET    | `/organizations/{orgId}/reservations/summary` | Get reservations summary          | -            | service_type     |
| GET    | `/organizations/{orgId}/reservations/all`     | Get all reservation opportunities | -            | service_type     |
| GET    | `/organizations/{orgId}/reservations`         | Get reservation recommendations   | -            | service_type     |

**Service Types**: `ec2`, `rds`, or omit for all services

### Schedules Module

**Base Path**: `/schedules`
**Guard**: AuthGuard (JWT Required)
**Controller**: `SchedulesController`

| Method | Endpoint                       | Description           | Request Body      | Query Parameters                            |
| ------ | ------------------------------ | --------------------- | ----------------- | ------------------------------------------- |
| POST   | `/schedules`                 | Create schedule       | CreateScheduleDto | -                                           |
| GET    | `/schedules`                 | List all schedules    | -                 | cloudAccountId, action, enabled, resourceId |
| GET    | `/schedules/{id}`            | Get schedule by ID    | -                 | -                                           |
| PATCH  | `/schedules/{id}`            | Update schedule       | UpdateScheduleDto | -                                           |
| DELETE | `/schedules/{id}`            | Delete schedule       | -                 | -                                           |
| POST   | `/schedules/{id}/enable`     | Enable schedule       | -                 | -                                           |
| POST   | `/schedules/{id}/disable`    | Disable schedule      | -                 | -                                           |
| POST   | `/schedules/{id}/execute`    | Execute schedule now  | -                 | -                                           |
| GET    | `/schedules/{id}/executions` | Get execution history | -                 | page, limit                                 |

### Sync Jobs Module

**Base Path**: `/sync-jobs`
**Guard**: AuthGuard (JWT Required)
**Controller**: `SyncJobsController`

| Method | Endpoint                                        | Description                | Request Body | Query Parameters |
| ------ | ----------------------------------------------- | -------------------------- | ------------ | ---------------- |
| GET    | `/sync-jobs/{jobId}`                          | Get sync job status        | -            | -                |
| GET    | `/sync-jobs/organization/{orgId}`             | Get organization sync jobs | -            | limit            |
| GET    | `/sync-jobs/cloud-account/{accountId}`        | Get account sync jobs      | -            | limit            |
| GET    | `/sync-jobs/cloud-account/{accountId}/latest` | Get latest sync job        | -            | -                |

### CloudTrail Sync Module

**Base Path**: `/cloudtrail-sync`
**Guard**: AuthGuard (JWT Required)
**Controller**: `CloudTrailSyncController`

| Method | Endpoint                                | Description                    | Request Body           | Query Parameters |
| ------ | --------------------------------------- | ------------------------------ | ---------------------- | ---------------- |
| POST   | `/cloudtrail-sync/sync`               | Sync CloudTrail events         | CloudTrailSyncDto      | -                |
| GET    | `/cloudtrail-sync/status/{accountId}` | Get CloudTrail sync status     | -                      | -                |
| POST   | `/cloudtrail-sync/sync-all`           | Sync all organization accounts | {hoursToSync?: number} | -                |

**CloudTrail Sync DTO:**

```typescript
{
  "cloudAccountId": "string",
  "hoursToSync": 24,
  "startTime": "2024-01-01T00:00:00Z",
  "endTime": "2024-01-01T23:59:59Z"
}
```

## Response Standards

### Success Response Format

```json
{
  "data": {}, // or []
  "message": "Operation completed successfully",
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

### Error Response Format

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "Detailed error description",
  "path": "/api/endpoint",
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

### Pagination Response Format

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "totalPages": 10
  },
  "metadata": {
    "total": 100
  }
}
```

## HTTP Status Codes

### Success Codes

- **200**: OK - Request successful
- **201**: Created - Resource created successfully
- **204**: No Content - Request successful, no content returned

### Client Error Codes

- **400**: Bad Request - Invalid request parameters
- **401**: Unauthorized - Authentication required
- **403**: Forbidden - Insufficient permissions
- **404**: Not Found - Resource not found
- **409**: Conflict - Resource conflict
- **422**: Unprocessable Entity - Validation errors

### Server Error Codes

- **500**: Internal Server Error - Generic server error
- **502**: Bad Gateway - Upstream service error
- **503**: Service Unavailable - Service temporarily unavailable

## Authentication & Authorization

### JWT Token Structure

```json
{
  "user": {
    "id": "user-uuid",
    "email": "user@example.com",
    "app_metadata": {},
    "user_metadata": {}
  }
}
```

### Organization Context

Organization access is controlled through:

1. URL path parameters (e.g., `/organizations/{orgId}/...`)
2. User membership validation in AuthGuard
3. Resource ownership verification in services

## Data Transfer Objects (DTOs)

### Common DTO Patterns

**Query DTOs** (for filtering and pagination):

```typescript
export class ResourceFilterDto {
  type?: string;
  region?: string;
  status?: string;
  tags?: Record<string, string>;
  created_after?: string;
  created_before?: string;
}
```

**Create DTOs** (for resource creation):

```typescript
export class CreateCloudAccountDto {
  name: string;
  description?: string;
  cloud_provider: 'aws';
  access_type: 'api_key' | 'role_arn';
  region: string;
  credentials: AwsCredentials;
}
```

**Update DTOs** (for resource updates):

```typescript
export class UpdateOrganizationDto {
  name?: string;
  description?: string;
}
```

## Background Processes

### Sync Jobs

- **Purpose**: Asynchronous resource discovery and cost data synchronization
- **Status Tracking**: Pending, Processing, Completed, Failed
- **Progress Reporting**: Percentage and descriptive messages

### CloudTrail Event Sync

- **Purpose**: Import AWS CloudTrail events for audit logging
- **Batch Processing**: Support for organization-wide sync operations
- **Time Range Configuration**: Flexible time range selection

### Scheduled Resource Control

- **Purpose**: Automated start/stop of AWS resources
- **Execution History**: Complete audit trail of scheduled actions
- **Cron-like Scheduling**: Flexible scheduling patterns

## Security Considerations

### Data Protection

1. **Credential Encryption**: AWS credentials stored encrypted in Vault service
2. **JWT Validation**: All endpoints protected by AuthGuard
3. **Organization Scoping**: Resources isolated by organization
4. **Audit Logging**: Comprehensive action tracking

### Rate Limiting

- Applied at the NestJS level for API protection
- AWS API rate limiting handled gracefully with retries

### Error Handling

- Sensitive information never exposed in error responses
- Detailed logging for debugging without security risks

## Performance Optimizations

### Async Operations

- Resource discovery runs asynchronously via sync jobs
- Cost data sync operations are non-blocking
- CloudTrail sync supports batch processing

### Database Optimizations

- Prisma ORM with optimized queries
- Indexed fields for common lookup patterns
- Connection pooling for high concurrency

### Caching Strategy

- Service-level caching for expensive operations
- AWS API response caching where appropriate

## Development Guidelines

### Adding New Endpoints

1. **Create Controller**: Implement in appropriate module with proper decorators
2. **Define DTOs**: Create request/response data transfer objects
3. **Implement Service**: Business logic in dedicated service class
4. **Add Guards**: Apply AuthGuard and any custom guards
5. **API Documentation**: Add Swagger decorators for automatic documentation
6. **Error Handling**: Implement proper error handling and status codes

### Testing API Endpoints

1. **Unit Tests**: Test controller methods with mocked services
2. **Integration Tests**: Test with real database connections
3. **Postman Collection**: Available in `/tests` directory
4. **Swagger Testing**: Interactive testing via `/api/docs`

## Monitoring & Observability

### Health Checks

- Database connectivity monitoring
- AWS service availability checks
- Background process status monitoring

### Logging Standards

- Structured logging with correlation IDs
- Different log levels for different environments
- Security-conscious logging (no sensitive data)

### Error Tracking

- Comprehensive error logging with stack traces
- Error categorization for monitoring systems
- Performance metrics tracking

## Deployment Considerations

### Environment Configuration

- Environment-specific configuration via .env files
- Database URL configuration for different environments
- AWS credentials management in production

### Database Migrations

- Prisma migration system for schema updates
- Automated migration deployment in CI/CD
- Database backup procedures before migrations

### Service Dependencies

- PostgreSQL database (via Supabase)
- AWS services for cloud resource management
- Vault service for credential encryption

## Compliance & Standards

### Data Privacy

- Minimal data collection principles
- Secure credential storage
- Audit trail for all operations

### Security Standards

- JWT-based authentication
- Organization-scoped authorization
- Encrypted sensitive data storage

### API Standards

- RESTful design principles
- Consistent response formats
- Proper HTTP status code usage
- OpenAPI/Swagger documentation
