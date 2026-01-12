# Backend Data Mutations

This document outlines the coding standards and patterns for data mutations in the CostPie NestJS backend application.

## Architecture Overview

Backend data mutations follow a layered MVC architecture pattern:

1. **Controllers** - Handle HTTP requests, routing, and validation
2. **Services** - Contain business logic and orchestrate operations
3. **DTOs** - Data Transfer Objects with validation decorators
4. **Prisma ORM** - Type-safe database operations and schema management
5. **Guards & Interceptors** - Authentication, authorization, and cross-cutting concerns

## Backend Data Flow Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   HTTP Client   │───▶│   NestJS Router  │───▶│   Controllers   │
│   (Frontend)    │    │   (app.module)   │    │ (@Controller)   │
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
│   HTTP Response │◀───│  Response Format │    │  DTOs + Pipes   │
│   (JSON/Error)  │    │   (Interceptors) │    │  (Validation)   │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       ▼
         │                       │              ┌─────────────────┐
         │                       │              │    Services     │
         │                       │              │ (Business Logic)│
         │                       │              └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  Error Handler  │    │  Swagger Docs    │    │   Prisma ORM    │
│  (Exception)    │    │  (API Schema)    │    │   (Database)    │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       ▼
         │                       │              ┌─────────────────┐
         │                       │              │   PostgreSQL    │
         │                       │              │   (Supabase)    │
         │                       │              └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │   Background Services    │
                    │  (Sync Jobs, Scheduler)  │
                    └──────────────────────────┘
```

## Technology Stack

- **Framework**: NestJS with TypeScript
- **Database ORM**: Prisma with PostgreSQL (Supabase)
- **Validation**: class-validator with class-transformer
- **Authentication**: JWT tokens via Supabase
- **Documentation**: Swagger/OpenAPI
- **Background Jobs**: Custom sync job system

## Backend MUST Follow NestJS Best Practices

### ✅ Current Architecture Standards:
- **Controllers for HTTP handling** - Route requests and coordinate responses
- **Services for business logic** - Encapsulate domain logic and data operations
- **DTOs with class-validator** - Type-safe validation for all inputs/outputs
- **Prisma for database operations** - Type-safe database access layer
- **Proper separation of concerns** - Clear layer boundaries and responsibilities

### Current Backend Architecture Pattern:
```
HTTP Client → NestJS Controller → Service Layer → Prisma ORM → Database
```

**Layer Responsibilities:**
- **Controllers**: Handle HTTP requests, validation, routing
- **Services**: Contain business logic and database operations
- **DTOs**: Use class-validator for validation (not Zod)
- **Prisma**: Type-safe database operations

## Module Organization

### File Structure Standards

Each feature MUST be organized as a self-contained NestJS module:

```typescript
// Example module structure
src/organizations/
├── dto/
│   ├── create-organization.dto.ts    // Input validation
│   ├── update-organization.dto.ts    // Update validation
│   └── index.ts                      // DTO exports
├── organizations.controller.ts        // HTTP endpoints
├── organizations.service.ts          // Business logic
├── organizations.module.ts           // Module definition
└── index.ts                         // Module exports
```

### Module Definition Pattern

Every module MUST follow this structure:

```typescript
// ✅ Correct - Proper module definition
@Module({
  imports: [PrismaModule, AuthModule],
  controllers: [OrganizationsController],
  providers: [OrganizationsService],
  exports: [OrganizationsService], // Export for use in other modules
})
export class OrganizationsModule {}

// ❌ Incorrect - Missing proper structure
@Module({
  controllers: [SomeController],
  // Missing providers, imports, or exports
})
export class BadModule {}
```

## Controllers Layer

### Controller Standards

Controllers MUST handle HTTP concerns only and delegate business logic to services:

```typescript
// ✅ Correct - Controller implementation
@ApiTags('organizations')
@Controller('organizations')
@UseGuards(AuthGuard)
@ApiBearerAuth()
export class OrganizationsController {
  constructor(private readonly organizationsService: OrganizationsService) {}

  @Post()
  @ApiOperation({ summary: 'Create a new organization' })
  @ApiResponse({ status: 201, description: 'Organization created successfully.' })
  @ApiResponse({ status: 400, description: 'Bad request.' })
  async create(
    @Body() createOrganizationDto: CreateOrganizationDto,
    @Request() req: AuthenticatedRequest
  ) {
    return await this.organizationsService.create(createOrganizationDto, req.user.id);
  }

  @Get('mine')
  @ApiOperation({ summary: 'Get user\'s organizations' })
  @ApiResponse({ status: 200, description: 'Organizations retrieved successfully.' })
  findUserOrganizations(@Request() req: AuthenticatedRequest) {
    return this.organizationsService.findUserOrganizations(req.user.id);
  }

  @Put(':orgId')
  @ApiOperation({ summary: 'Update an organization' })
  @ApiResponse({ status: 200, description: 'Organization updated successfully.' })
  @ApiResponse({ status: 404, description: 'Organization not found.' })
  update(
    @Param('orgId') id: string,
    @Body() updateOrganizationDto: UpdateOrganizationDto,
  ) {
    return this.organizationsService.update(id, updateOrganizationDto);
  }
}

// ❌ Incorrect - Business logic in controller
@Controller('organizations')
export class BadController {
  constructor(private prisma: PrismaService) {} // Don't inject Prisma directly
  
  @Post()
  async create(@Body() data: any) { // No DTO validation
    // Don't put business logic here
    const org = await this.prisma.organization.create({
      data: { ...data, createdAt: new Date() }
    });
    return org;
  }
}
```

### HTTP Decorators and Status Codes

Use proper HTTP decorators and status codes:

```typescript
// ✅ Correct - Proper HTTP methods and status codes
@Post()                    // 201 Created
@Get()                     // 200 OK
@Put(':id')               // 200 OK
@Patch(':id')             // 200 OK
@Delete(':id')            // 200 OK or 204 No Content
@HttpCode(HttpStatus.NO_CONTENT)
async remove(@Param('id') id: string) {
  await this.service.remove(id);
  // No return for 204 status
}

// ✅ Correct - Custom status codes when needed
@Post(':id/sync')
@HttpCode(HttpStatus.ACCEPTED) // 202 Accepted for async operations
async triggerSync(@Param('id') id: string) {
  const job = await this.service.createSyncJob(id);
  return { jobId: job.id, status: 'queued' };
}
```

### Request/Response Interceptors

Use interceptors for cross-cutting concerns:

```typescript
// ✅ Correct - Response transformation interceptor
@Injectable()
export class ResponseInterceptor<T> implements NestInterceptor<T, any> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        data,
        message: 'Success',
        timestamp: new Date().toISOString(),
      })),
      catchError(error => {
        // Log error with context
        const request = context.switchToHttp().getRequest();
        console.error(`Error in ${request.method} ${request.url}:`, error);
        throw error;
      })
    );
  }
}

// Apply globally or per controller
@UseInterceptors(ResponseInterceptor)
@Controller('organizations')
export class OrganizationsController {}
```

## Services Layer

### Service Standards

Services MUST contain business logic and coordinate data operations:

```typescript
// ✅ Correct - Service implementation
@Injectable()
export class OrganizationsService {
  constructor(private prisma: PrismaService) {}

  async create(createOrganizationDto: CreateOrganizationDto, userId: string) {
    try {
      // Validate business rules
      await this.validateUserCanCreateOrganization(userId);
      
      // Perform database operation
      const organization = await this.prisma.organization.create({
        data: {
          ...createOrganizationDto,
          owner_user_id: userId,
          created_at: new Date(),
          updated_at: new Date(),
        },
      });

      // Post-creation business logic
      await this.initializeDefaultSettings(organization.id);
      
      return organization;
    } catch (error) {
      // Convert database errors to business exceptions
      if (error.code === 'P2002') {
        throw new ConflictException('Organization name already exists');
      }
      throw new InternalServerErrorException('Failed to create organization');
    }
  }

  async findUserOrganizations(userId: string) {
    // Input validation
    if (!userId) {
      throw new BadRequestException('User ID is required');
    }

    try {
      return await this.prisma.organization.findMany({
        where: {
          owner_user_id: userId,
        },
        include: {
          cloud_accounts: {
            select: { id: true, name: true, cloud_provider: true }
          }
        },
        orderBy: { created_at: 'desc' },
      });
    } catch (error) {
      console.error('Failed to find user organizations:', error);
      throw new InternalServerErrorException('Failed to retrieve organizations');
    }
  }

  async update(id: string, updateOrganizationDto: UpdateOrganizationDto) {
    // Check if organization exists
    const existingOrg = await this.findOne(id);
    if (!existingOrg) {
      throw new NotFoundException('Organization not found');
    }

    try {
      return await this.prisma.organization.update({
        where: { id },
        data: {
          ...updateOrganizationDto,
          updated_at: new Date(),
        },
      });
    } catch (error) {
      throw new InternalServerErrorException('Failed to update organization');
    }
  }

  // Private helper methods for business logic
  private async validateUserCanCreateOrganization(userId: string): Promise<void> {
    const existingOrgs = await this.prisma.organization.count({
      where: { owner_user_id: userId }
    });
    
    if (existingOrgs >= 5) { // Business rule example
      throw new ForbiddenException('User cannot create more than 5 organizations');
    }
  }

  private async initializeDefaultSettings(organizationId: string): Promise<void> {
    await this.prisma.organizationSettings.create({
      data: {
        organization_id: organizationId,
        notification_email: true,
        cost_alerts_enabled: true,
      }
    });
  }
}

// ❌ Incorrect - No error handling or business logic
@Injectable()
export class BadService {
  constructor(private prisma: PrismaService) {}

  async create(data: any) { // No typing
    return this.prisma.organization.create({ data }); // No error handling
  }
}
```

### Transaction Management

Use Prisma transactions for complex operations:

```typescript
// ✅ Correct - Transaction usage
@Injectable()
export class CloudAccountsService {
  constructor(private prisma: PrismaService) {}

  async createWithInitialSetup(
    organizationId: string, 
    createCloudAccountDto: CreateCloudAccountDto
  ) {
    return await this.prisma.$transaction(async (prisma) => {
      // Create cloud account
      const cloudAccount = await prisma.cloudAccount.create({
        data: {
          ...createCloudAccountDto,
          organization_id: organizationId,
        }
      });

      // Create initial sync job
      const syncJob = await prisma.syncJob.create({
        data: {
          cloud_account_id: cloudAccount.id,
          organization_id: organizationId,
          job_type: 'INITIAL_DISCOVERY',
          status: 'PENDING',
        }
      });

      // Log audit event
      await prisma.auditLog.create({
        data: {
          organization_id: organizationId,
          action_type: 'CREATE',
          target_resource_type: 'CLOUD_ACCOUNT',
          target_resource_id: cloudAccount.id,
          details: { cloudAccount, syncJob },
        }
      });

      return { cloudAccount, syncJob };
    });
  }
}
```

## DTOs and Validation

### DTO Standards

Use class-validator decorators for comprehensive validation:

```typescript
// ✅ Correct - Comprehensive DTO validation
import { IsNotEmpty, IsString, IsOptional, IsEnum, IsEmail, Length } from 'class-validator';
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';

export class CreateOrganizationDto {
  @ApiProperty({
    description: 'The name of the organization',
    example: 'Acme Corporation',
    minLength: 1,
    maxLength: 100,
  })
  @IsString()
  @IsNotEmpty()
  @Length(1, 100)
  name: string;

  @ApiPropertyOptional({
    description: 'The subscription plan for the organization',
    example: 'free',
    default: 'free',
    enum: ['free', 'pro', 'enterprise'],
  })
  @IsString()
  @IsOptional()
  @IsEnum(['free', 'pro', 'enterprise'])
  subscription_plan?: string;

  @ApiPropertyOptional({
    description: 'Contact email for the organization',
    example: 'contact@acme.com',
  })
  @IsEmail()
  @IsOptional()
  contact_email?: string;
}

// ✅ Correct - Update DTO extending create DTO
export class UpdateOrganizationDto extends PartialType(CreateOrganizationDto) {}

// ❌ Incorrect - No validation
export class BadDto {
  name: any; // No validation, no typing
  email: string; // No validation decorators
}
```

### Nested Object Validation

Handle complex nested validation:

```typescript
// ✅ Correct - Nested validation
export class CreateCloudAccountDto {
  @ApiProperty({ description: 'Account name' })
  @IsString()
  @IsNotEmpty()
  name: string;

  @ApiProperty({ description: 'Cloud provider', enum: ['aws'] })
  @IsEnum(['aws'])
  cloud_provider: 'aws';

  @ApiProperty({ description: 'AWS credentials' })
  @ValidateNested()
  @Type(() => AwsCredentialsDto)
  credentials: AwsCredentialsDto;
}

export class AwsCredentialsDto {
  @ApiProperty({ description: 'AWS Access Key ID' })
  @IsString()
  @IsNotEmpty()
  @Matches(/^AKIA[0-9A-Z]{16}$/, {
    message: 'Invalid AWS Access Key ID format'
  })
  accessKeyId: string;

  @ApiProperty({ description: 'AWS Secret Access Key' })
  @IsString()
  @IsNotEmpty()
  @MinLength(40)
  secretAccessKey: string;

  @ApiProperty({ description: 'AWS Region' })
  @IsString()
  @IsNotEmpty()
  region: string;
}
```

### Custom Validation Decorators

Create reusable validation decorators:

```typescript
// ✅ Correct - Custom validator
import { registerDecorator, ValidationOptions, ValidationArguments } from 'class-validator';

export function IsValidAwsRegion(validationOptions?: ValidationOptions) {
  return function (object: Object, propertyName: string) {
    registerDecorator({
      name: 'isValidAwsRegion',
      target: object.constructor,
      propertyName: propertyName,
      options: validationOptions,
      validator: {
        validate(value: any, args: ValidationArguments) {
          const validRegions = [
            'us-east-1', 'us-west-1', 'us-west-2', 'eu-west-1', 
            'eu-central-1', 'ap-southeast-1', 'ap-northeast-1'
          ];
          return typeof value === 'string' && validRegions.includes(value);
        },
        defaultMessage(args: ValidationArguments) {
          return 'Invalid AWS region';
        },
      },
    });
  };
}

// Usage in DTO
export class CreateCloudAccountDto {
  @ApiProperty({ description: 'AWS region' })
  @IsString()
  @IsValidAwsRegion()
  region: string;
}
```

## Database Operations with Prisma

### Prisma Best Practices

Follow type-safe database operation patterns:

```typescript
// ✅ Correct - Prisma operations with proper typing
@Injectable()
export class ResourcesService {
  constructor(private prisma: PrismaService) {}

  async findByCloudAccount(
    accountId: string, 
    filters: ResourceFilterDto
  ): Promise<Resource[]> {
    const whereClause: Prisma.ResourceWhereInput = {
      cloud_account_id: accountId,
      ...(filters.type && { resource_type: filters.type }),
      ...(filters.region && { region: filters.region }),
      ...(filters.status && { status: filters.status }),
      ...(filters.created_after && {
        discovered_at: { gte: new Date(filters.created_after) }
      }),
      ...(filters.created_before && {
        discovered_at: { lte: new Date(filters.created_before) }
      }),
    };

    return await this.prisma.resource.findMany({
      where: whereClause,
      include: {
        cost_entries: {
          where: {
            date: {
              gte: startOfMonth(new Date()),
              lte: endOfMonth(new Date()),
            }
          },
          orderBy: { date: 'desc' },
          take: 30,
        },
        tags: true,
        team: {
          select: { id: true, name: true }
        }
      },
      orderBy: { discovered_at: 'desc' },
    });
  }

  async bulkUpdate(updates: BulkResourceUpdateDto[]): Promise<number> {
    const updatePromises = updates.map(update =>
      this.prisma.resource.update({
        where: { id: update.id },
        data: {
          status: update.status,
          updated_at: new Date(),
          ...(update.team_id && { team_id: update.team_id }),
        }
      })
    );

    const results = await Promise.allSettled(updatePromises);
    const successCount = results.filter(result => result.status === 'fulfilled').length;
    
    return successCount;
  }

  // Use upsert for idempotent operations
  async syncResourceFromAws(resourceData: AwsResourceData): Promise<Resource> {
    return await this.prisma.resource.upsert({
      where: { 
        aws_resource_id_cloud_account_id: {
          aws_resource_id: resourceData.id,
          cloud_account_id: resourceData.cloudAccountId,
        }
      },
      update: {
        name: resourceData.name,
        status: resourceData.status,
        configuration: resourceData.configuration,
        last_seen_at: new Date(),
        updated_at: new Date(),
      },
      create: {
        aws_resource_id: resourceData.id,
        cloud_account_id: resourceData.cloudAccountId,
        name: resourceData.name,
        resource_type: resourceData.type,
        region: resourceData.region,
        status: resourceData.status,
        configuration: resourceData.configuration,
        discovered_at: new Date(),
        last_seen_at: new Date(),
      },
    });
  }
}
```

### Raw Queries and Performance

Use raw queries for complex operations:

```typescript
// ✅ Correct - Raw query for complex analytics
@Injectable()
export class CostsService {
  constructor(private prisma: PrismaService) {}

  async getCostTrendWithAggregation(
    organizationId: string,
    params: CostTrendParamsDto
  ): Promise<CostTrendData[]> {
    // Use raw query for complex aggregations
    const result = await this.prisma.$queryRaw<CostTrendData[]>`
      SELECT 
        DATE_TRUNC('day', date) as period,
        SUM(amount) as total_cost,
        COUNT(DISTINCT resource_id) as resource_count,
        AVG(amount) as avg_cost_per_resource
      FROM cost_entries ce
      JOIN resources r ON ce.resource_id = r.id
      JOIN cloud_accounts ca ON r.cloud_account_id = ca.id
      WHERE ca.organization_id = ${organizationId}
        AND ce.date >= ${params.start_date}
        AND ce.date <= ${params.end_date}
      GROUP BY DATE_TRUNC('day', date)
      ORDER BY period ASC
    `;

    return result.map(row => ({
      ...row,
      total_cost: Number(row.total_cost),
      avg_cost_per_resource: Number(row.avg_cost_per_resource),
    }));
  }
}
```

## Error Handling and Exceptions

### Exception Standards

Use NestJS built-in exceptions and create custom ones when needed:

```typescript
// ✅ Correct - Proper exception handling
@Injectable()
export class CloudAccountsService {
  constructor(private prisma: PrismaService) {}

  async findOne(id: string): Promise<CloudAccount> {
    if (!id) {
      throw new BadRequestException('Cloud account ID is required');
    }

    try {
      const cloudAccount = await this.prisma.cloudAccount.findUnique({
        where: { id },
        include: { organization: true }
      });

      if (!cloudAccount) {
        throw new NotFoundException(`Cloud account with ID ${id} not found`);
      }

      return cloudAccount;
    } catch (error) {
      if (error instanceof NotFoundException) {
        throw error; // Re-throw known exceptions
      }
      
      // Log unexpected errors
      console.error(`Failed to find cloud account ${id}:`, error);
      throw new InternalServerErrorException('Failed to retrieve cloud account');
    }
  }

  async validateCredentials(credentials: AwsCredentialsDto): Promise<boolean> {
    try {
      // AWS SDK validation logic
      const isValid = await this.awsService.validateCredentials(credentials);
      return isValid;
    } catch (error) {
      if (error.code === 'InvalidUserID.NotFound') {
        throw new UnauthorizedException('Invalid AWS credentials');
      }
      if (error.code === 'AccessDenied') {
        throw new ForbiddenException('AWS credentials lack required permissions');
      }
      
      throw new BadRequestException('Failed to validate AWS credentials');
    }
  }
}

// ✅ Correct - Custom exception classes
export class CloudAccountNotSyncedException extends BadRequestException {
  constructor(accountId: string) {
    super(`Cloud account ${accountId} has not been synced yet`);
  }
}

export class ResourceLimitExceededException extends ForbiddenException {
  constructor(limit: number) {
    super(`Resource limit of ${limit} exceeded for this organization`);
  }
}
```

### Global Exception Filter

Implement global exception handling:

```typescript
// ✅ Correct - Global exception filter
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal server error';
    let error = 'Internal Server Error';

    if (exception instanceof HttpException) {
      status = exception.getStatus();
      const exceptionResponse = exception.getResponse();
      
      if (typeof exceptionResponse === 'object') {
        message = (exceptionResponse as any).message || exception.message;
        error = (exceptionResponse as any).error || exception.name;
      } else {
        message = exceptionResponse as string;
        error = exception.name;
      }
    } else if (exception instanceof Prisma.PrismaClientKnownRequestError) {
      // Handle Prisma-specific errors
      status = HttpStatus.BAD_REQUEST;
      message = this.handlePrismaError(exception);
      error = 'Database Error';
    }

    // Log error for monitoring
    console.error(`Error ${status} in ${request.method} ${request.url}:`, {
      exception: exception instanceof Error ? exception.message : exception,
      stack: exception instanceof Error ? exception.stack : undefined,
      user: (request as any).user?.id,
      organizationId: request.headers['x-organization-id'],
    });

    response.status(status).json({
      statusCode: status,
      error,
      message,
      path: request.url,
      timestamp: new Date().toISOString(),
    });
  }

  private handlePrismaError(error: Prisma.PrismaClientKnownRequestError): string {
    switch (error.code) {
      case 'P2002':
        return 'A record with this information already exists';
      case 'P2025':
        return 'Record not found';
      case 'P2003':
        return 'Invalid reference to related record';
      default:
        return 'Database operation failed';
    }
  }
}

// Apply globally in main.ts
async function bootstrap() {
  const app = await NestJSFactory.create(AppModule);
  app.useGlobalFilters(new AllExceptionsFilter());
  await app.listen(3001);
}
```

## Authentication and Authorization

### Auth Guard Implementation

Implement comprehensive authentication and authorization:

```typescript
// ✅ Correct - Auth guard with organization context
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private supabaseService: SupabaseService,
    private usersService: UsersService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractTokenFromHeader(request);

    if (!token) {
      throw new UnauthorizedException('Authentication token is required');
    }

    try {
      // Verify JWT token with Supabase
      const { data: user, error } = await this.supabaseService
        .getClient()
        .auth.getUser(token);

      if (error || !user) {
        throw new UnauthorizedException('Invalid authentication token');
      }

      // Attach user to request
      request.user = {
        id: user.user.id,
        email: user.user.email,
        app_metadata: user.user.app_metadata,
        user_metadata: user.user.user_metadata,
      };

      // Validate organization access if organization ID is in headers
      const organizationId = request.headers['x-organization-id'];
      if (organizationId) {
        await this.validateOrganizationAccess(user.user.id, organizationId);
        request.organizationId = organizationId;
      }

      return true;
    } catch (error) {
      if (error instanceof UnauthorizedException) {
        throw error;
      }
      throw new UnauthorizedException('Authentication failed');
    }
  }

  private extractTokenFromHeader(request: Request): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }

  private async validateOrganizationAccess(
    userId: string, 
    organizationId: string
  ): Promise<void> {
    const hasAccess = await this.usersService.hasOrganizationAccess(
      userId, 
      organizationId
    );
    
    if (!hasAccess) {
      throw new ForbiddenException('Access to this organization is not allowed');
    }
  }
}
```

### Role-Based Access Control

Implement role-based permissions:

```typescript
// ✅ Correct - Role-based decorator and guard
export const Roles = (...roles: UserRole[]) => SetMetadata('roles', roles);

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<UserRole[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles) {
      return true; // No role restriction
    }

    const { user, organizationId } = context.switchToHttp().getRequest();
    
    // Check user's role in the organization
    const userRole = this.getUserRoleInOrganization(user.id, organizationId);
    
    return requiredRoles.some(role => role === userRole);
  }

  private getUserRoleInOrganization(userId: string, organizationId: string): UserRole {
    // Implementation to get user role from database
    // Return user's role in the specific organization
  }
}

// Usage in controllers
@Controller('organizations')
@UseGuards(AuthGuard, RolesGuard)
export class OrganizationsController {
  @Post()
  @Roles(UserRole.ADMIN, UserRole.OWNER)
  create(@Body() dto: CreateOrganizationDto) {
    // Only admins and owners can create organizations
  }

  @Delete(':id')
  @Roles(UserRole.OWNER)
  remove(@Param('id') id: string) {
    // Only owners can delete organizations
  }
}
```

## Background Jobs and Async Operations

### Async Job Patterns

Implement background job processing:

```typescript
// ✅ Correct - Async job service
@Injectable()
export class SyncJobsService {
  constructor(
    private prisma: PrismaService,
    private awsService: AwsService,
  ) {}

  async createSyncJob(
    cloudAccountId: string,
    organizationId: string,
    jobType: SyncJobType,
  ): Promise<SyncJob> {
    const syncJob = await this.prisma.syncJob.create({
      data: {
        cloud_account_id: cloudAccountId,
        organization_id: organizationId,
        job_type: jobType,
        status: 'PENDING',
        progress: 0,
        created_at: new Date(),
      },
    });

    // Queue job for background processing
    setImmediate(() => this.processSyncJob(syncJob.id));
    
    return syncJob;
  }

  async processSyncJob(jobId: string): Promise<void> {
    try {
      const job = await this.prisma.syncJob.findUnique({
        where: { id: jobId },
        include: { cloud_account: true },
      });

      if (!job) {
        console.error(`Sync job ${jobId} not found`);
        return;
      }

      // Update status to processing
      await this.updateSyncJobStatus(jobId, 'PROCESSING', 10, 'Starting sync');

      // Perform the actual sync operation
      const result = await this.performSync(job);

      // Update with completion status
      await this.updateSyncJobStatus(
        jobId, 
        'COMPLETED', 
        100, 
        `Sync completed. Processed ${result.resourceCount} resources`
      );

    } catch (error) {
      console.error(`Sync job ${jobId} failed:`, error);
      
      await this.updateSyncJobStatus(
        jobId,
        'FAILED',
        0,
        'Sync failed',
        error.message
      );
    }
  }

  async updateSyncJobStatus(
    jobId: string,
    status: SyncJobStatus,
    progress: number,
    progressMessage: string,
    errorMessage?: string,
  ): Promise<void> {
    await this.prisma.syncJob.update({
      where: { id: jobId },
      data: {
        status,
        progress,
        progress_message: progressMessage,
        error_message: errorMessage,
        updated_at: new Date(),
        ...(status === 'COMPLETED' && { completed_at: new Date() }),
      },
    });
  }

  private async performSync(job: SyncJob): Promise<{ resourceCount: number }> {
    // Implementation of actual sync logic
    const resources = await this.awsService.discoverResources(job.cloud_account_id);
    
    // Batch upsert resources
    await this.batchUpsertResources(resources);
    
    return { resourceCount: resources.length };
  }
}
```

## Testing Patterns

### Unit Testing Services

Write comprehensive unit tests for services:

```typescript
// ✅ Correct - Service unit tests
describe('OrganizationsService', () => {
  let service: OrganizationsService;
  let prisma: PrismaService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        OrganizationsService,
        {
          provide: PrismaService,
          useValue: {
            organization: {
              create: jest.fn(),
              findMany: jest.fn(),
              findUnique: jest.fn(),
              update: jest.fn(),
              count: jest.fn(),
            },
          },
        },
      ],
    }).compile();

    service = module.get<OrganizationsService>(OrganizationsService);
    prisma = module.get<PrismaService>(PrismaService);
  });

  describe('create', () => {
    it('should create an organization successfully', async () => {
      const createDto: CreateOrganizationDto = {
        name: 'Test Organization',
        subscription_plan: 'free',
      };
      const userId = 'user123';
      const mockOrganization = { id: 'org123', ...createDto, owner_user_id: userId };

      jest.spyOn(prisma.organization, 'create').mockResolvedValue(mockOrganization as any);

      const result = await service.create(createDto, userId);

      expect(prisma.organization.create).toHaveBeenCalledWith({
        data: {
          ...createDto,
          owner_user_id: userId,
          created_at: expect.any(Date),
          updated_at: expect.any(Date),
        },
      });
      expect(result).toEqual(mockOrganization);
    });

    it('should throw ConflictException for duplicate organization name', async () => {
      const createDto: CreateOrganizationDto = { name: 'Duplicate Org' };
      const userId = 'user123';

      jest.spyOn(prisma.organization, 'create').mockRejectedValue({
        code: 'P2002', // Prisma unique constraint violation
      });

      await expect(service.create(createDto, userId))
        .rejects
        .toThrow(ConflictException);
    });
  });
});
```

### Integration Testing Controllers

Test controllers with proper mocking:

```typescript
// ✅ Correct - Controller integration tests
describe('OrganizationsController', () => {
  let app: INestApplication;
  let organizationsService: OrganizationsService;

  beforeEach(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      controllers: [OrganizationsController],
      providers: [
        {
          provide: OrganizationsService,
          useValue: {
            create: jest.fn(),
            findUserOrganizations: jest.fn(),
            update: jest.fn(),
          },
        },
      ],
    })
    .overrideGuard(AuthGuard)
    .useValue({ canActivate: () => true }) // Mock auth guard
    .compile();

    app = moduleFixture.createNestApplication();
    organizationsService = moduleFixture.get<OrganizationsService>(OrganizationsService);
    
    await app.init();
  });

  describe('POST /organizations', () => {
    it('should create an organization', async () => {
      const createDto = { name: 'Test Org' };
      const mockOrganization = { id: 'org123', ...createDto };

      jest.spyOn(organizationsService, 'create').mockResolvedValue(mockOrganization as any);

      return request(app.getHttpServer())
        .post('/organizations')
        .send(createDto)
        .expect(201)
        .expect(res => {
          expect(res.body).toMatchObject(mockOrganization);
        });
    });

    it('should return 400 for invalid input', async () => {
      const invalidDto = { name: '' }; // Empty name

      return request(app.getHttpServer())
        .post('/organizations')
        .send(invalidDto)
        .expect(400);
    });
  });
});
```

## Performance and Monitoring

### Database Query Optimization

Optimize database queries and add monitoring:

```typescript
// ✅ Correct - Optimized queries with monitoring
@Injectable()
export class ResourcesService {
  constructor(
    private prisma: PrismaService,
    private logger: Logger,
  ) {}

  async findWithPagination(
    accountId: string,
    filters: ResourceFilterDto,
    pagination: PaginationDto,
  ): Promise<PaginatedResponse<Resource>> {
    const startTime = Date.now();
    
    try {
      const whereClause = this.buildWhereClause(accountId, filters);
      
      // Get total count and data in parallel
      const [total, resources] = await Promise.all([
        this.prisma.resource.count({ where: whereClause }),
        this.prisma.resource.findMany({
          where: whereClause,
          include: {
            cost_entries: {
              where: { date: { gte: startOfMonth(new Date()) } },
              orderBy: { date: 'desc' },
              take: 10, // Limit cost entries
            },
            tags: { select: { key: true, value: true } }, // Only needed fields
          },
          skip: (pagination.page - 1) * pagination.limit,
          take: pagination.limit,
          orderBy: { discovered_at: 'desc' },
        }),
      ]);

      const duration = Date.now() - startTime;
      this.logger.log(`Query completed in ${duration}ms, returned ${resources.length} resources`);

      return {
        data: resources,
        pagination: {
          page: pagination.page,
          limit: pagination.limit,
          total,
          totalPages: Math.ceil(total / pagination.limit),
        },
      };
    } catch (error) {
      const duration = Date.now() - startTime;
      this.logger.error(`Query failed after ${duration}ms`, error);
      throw error;
    }
  }

  private buildWhereClause(accountId: string, filters: ResourceFilterDto): Prisma.ResourceWhereInput {
    return {
      cloud_account_id: accountId,
      ...(filters.type && { resource_type: filters.type }),
      ...(filters.region && { region: filters.region }),
      ...(filters.status && { status: filters.status }),
      ...(filters.tags && {
        tags: {
          some: {
            key: filters.tags.key,
            value: filters.tags.value,
          },
        },
      }),
    };
  }
}
```

## Best Practices Summary

### ✅ DO:
1. **Use NestJS decorators** for controllers, services, and validation
2. **Implement proper error handling** with custom exceptions
3. **Validate all inputs** with class-validator DTOs
4. **Use Prisma transactions** for complex operations
5. **Implement authentication/authorization** with guards
6. **Write comprehensive tests** for services and controllers
7. **Monitor performance** with logging and metrics
8. **Handle async operations** with proper job queuing
9. **Document APIs** with Swagger decorators
10. **Follow SOLID principles** in service design

### ❌ DON'T:
1. **Put business logic in controllers** - keep controllers thin
2. **Inject Prisma directly in controllers** - use services
3. **Skip validation** - always validate DTOs
4. **Ignore error handling** - handle all error cases
5. **Use raw database queries unnecessarily** - prefer Prisma methods
6. **Skip authentication** - protect all endpoints
7. **Forget to test** - maintain good test coverage
8. **Block the main thread** - use async patterns for long operations
9. **Hardcode values** - use configuration and environment variables
10. **Skip logging** - log important operations and errors

## Migration and Maintenance

### Database Migrations

Handle schema changes properly:

```typescript
// ✅ Correct - Migration-safe code
@Injectable()
export class MigrationSafeService {
  constructor(private prisma: PrismaService) {}

  async safeUpdate(id: string, data: UpdateDto): Promise<Resource> {
    // Check if new fields exist before using them
    const schemaFields = await this.getAvailableFields();
    
    const updateData: any = { ...data };
    
    // Only include fields that exist in current schema
    if (!schemaFields.includes('new_field')) {
      delete updateData.new_field;
    }

    return await this.prisma.resource.update({
      where: { id },
      data: updateData,
    });
  }

  private async getAvailableFields(): Promise<string[]> {
    // Query information schema or use Prisma introspection
    // to get available fields dynamically
  }
}
```

This backend data mutation pattern ensures type safety, proper error handling, optimal performance, and excellent maintainability while following NestJS best practices and maintaining clean separation of concerns across all layers.