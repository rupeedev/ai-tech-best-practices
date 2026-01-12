# Data Fetching Guidelines

## Architecture Overview

CostPie follows a **React/Vite frontend + NestJS backend architecture** with client-side data fetching patterns using modern React best practices.

### Current Tech Stack
- **Frontend**: React 18 + Vite + TypeScript
- **State Management**: TanStack Query for server state
- **Backend**: NestJS with PostgreSQL (Supabase)
- **Authentication**: Supabase JWT tokens
- **Form Handling**: React Hook Form + Zod validation

## Core Data Fetching Principles

### ✅ Required Architecture Pattern
```
React Component → API Client → HTTP Request → NestJS Backend → Database
```

### ✅ Approved Data Fetching Methods
- **TanStack Query hooks** - Primary method for server state management
- **API Client functions** - Centralized HTTP communication layer
- **React Hook Form** - Form data handling with validation
- **Custom hooks** - Reusable query and mutation logic

### ❌ Prohibited Patterns
- Direct fetch/axios calls in components
- Manual state management for server data (useState + useEffect)
- Untyped API calls
- Bypassing the centralized API client
- Client-side data mutations without proper error handling

## Security Standards

### Authentication & Authorization

**CRITICAL SECURITY REQUIREMENTS**:

#### 1. JWT Token Management
```typescript
// ✅ Correct - Automatic token injection via API client
const apiClient = {
  headers: {
    'Authorization': `Bearer ${supabaseAuth.session?.access_token}`,
    'x-organization-id': currentOrganization.id
  }
};

// ❌ Incorrect - Manual token handling
fetch('/api/data', {
  headers: { 'Authorization': `Bearer ${token}` } // Don't manage manually
});
```

#### 2. Organization-Based Data Access
- **Organization Scoping**: All data requests MUST include organization context
- **User Isolation**: Users can ONLY access data within their organization
- **Role Validation**: Backend enforces role-based permissions
- **Resource Ownership**: Validate ownership before any data operations

```typescript
// ✅ Correct - Organization-scoped request
export const resourcesApi = {
  async getByOrganization(orgId: string): Promise<Resource[]> {
    return apiClient.get<Resource[]>(`/organizations/${orgId}/resources`);
  }
};

// ❌ Incorrect - Global resource access
export const badResourcesApi = {
  async getAll(): Promise<Resource[]> {
    return apiClient.get<Resource[]>('/resources'); // No org scoping
  }
};
```

#### 3. Sensitive Data Protection
- **Credentials Encryption**: AWS credentials encrypted at rest and in transit
- **PII Minimization**: Only request necessary user data
- **Error Sanitization**: Never expose sensitive data in error messages
- **Token Refresh**: Automatic token refresh via Supabase auth

```typescript
// ✅ Correct - Safe error handling
const handleApiError = (error: ApiError) => {
  // Log full error details for monitoring (server-side only)
  errorLogger.logError(error, { context: 'API_CALL' });
  
  // Show sanitized error to user
  const userMessage = error.status === 403 
    ? 'You do not have permission to perform this action'
    : 'An error occurred. Please try again.';
    
  return userMessage;
};

// ❌ Incorrect - Exposing sensitive data
const badErrorHandler = (error: ApiError) => {
  alert(error.message); // Might contain sensitive info
};
```

### Input Validation & Sanitization

#### 1. Client-Side Validation
```typescript
// ✅ Correct - Comprehensive Zod validation
const cloudAccountSchema = z.object({
  name: z.string()
    .min(1, 'Account name is required')
    .max(100, 'Account name must be less than 100 characters')
    .regex(/^[a-zA-Z0-9\s\-_]+$/, 'Account name contains invalid characters'),
  credentials: z.object({
    accessKeyId: z.string()
      .regex(/^AKIA[0-9A-Z]{16}$/, 'Invalid AWS Access Key ID format'),
    secretAccessKey: z.string()
      .min(40, 'Invalid AWS Secret Access Key length')
  })
});
```

#### 2. XSS Prevention
- **Content Sanitization**: All user input sanitized before display
- **CSP Headers**: Content Security Policy enforced
- **Safe Rendering**: Use React's built-in XSS protection

```typescript
// ✅ Correct - Safe content rendering
function ResourceCard({ resource }: { resource: Resource }) {
  return (
    <div>
      {/* React automatically escapes content */}
      <h3>{resource.name}</h3>
      <p>{resource.description}</p>
    </div>
  );
}

// ❌ Incorrect - Dangerous HTML injection
function BadResourceCard({ resource }: { resource: Resource }) {
  return (
    <div dangerouslySetInnerHTML={{ __html: resource.content }} />
  );
}
```

## Performance Optimization

### Query Optimization Strategies

#### 1. Intelligent Caching
```typescript
// ✅ Correct - Optimized query configuration
export function useResources(accountId: string, filters?: ResourceFilters) {
  return useQuery({
    queryKey: ['resources', accountId, filters],
    queryFn: () => resourcesApi.getByAccount(accountId, filters),
    enabled: !!accountId, // Only run if accountId exists
    staleTime: 2 * 60 * 1000, // 2 minutes
    gcTime: 5 * 60 * 1000, // 5 minutes garbage collection
    refetchOnWindowFocus: false,
    select: (data) => {
      // Transform data to reduce memory usage
      return data.map(resource => ({
        id: resource.id,
        name: resource.name,
        type: resource.type,
        status: resource.status,
        // Only include needed fields
      }));
    },
  });
}
```

#### 2. Request Deduplication & Batching
```typescript
// ✅ Correct - Automatic deduplication via TanStack Query
function MultipleResourceComponents() {
  const query1 = useResources(accountId); // Same query key
  const query2 = useResources(accountId); // Automatically deduplicated
  const query3 = useResources(accountId); // Single network request
  
  // All components receive the same cached data
}

// ✅ Correct - Batch operations for multiple resources
export function useBatchResourceOperations() {
  return useMutation({
    mutationFn: async (operations: ResourceOperation[]) => {
      // Batch multiple operations in single API call
      return resourcesApi.batchUpdate(operations);
    },
    onSuccess: () => {
      // Invalidate relevant queries efficiently
      queryClient.invalidateQueries({ queryKey: ['resources'] });
    },
  });
}
```

#### 3. Optimistic Updates for UX
```typescript
// ✅ Correct - Optimistic updates with rollback
export function useOptimisticResourceUpdate(resourceId: string) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (updates: Partial<Resource>) => 
      resourcesApi.update(resourceId, updates),
    onMutate: async (updates) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries(['resources', resourceId]);
      
      // Snapshot current value
      const previousResource = queryClient.getQueryData(['resources', resourceId]);
      
      // Optimistically update
      queryClient.setQueryData(['resources', resourceId], (old: Resource) => ({
        ...old,
        ...updates,
      }));
      
      return { previousResource };
    },
    onError: (err, updates, context) => {
      // Rollback on error
      queryClient.setQueryData(['resources', resourceId], context?.previousResource);
    },
    onSettled: () => {
      // Refetch to ensure consistency
      queryClient.invalidateQueries(['resources', resourceId]);
    },
  });
}
```

### Bundle Optimization

#### 1. Code Splitting & Lazy Loading
```typescript
// ✅ Correct - Route-based code splitting
import { lazy, Suspense } from 'react';

const CloudAccounts = lazy(() => import('@/pages/CloudAccounts'));
const Resources = lazy(() => import('@/pages/Resources'));
const CostManagement = lazy(() => import('@/pages/CostManagement'));

function AppRouter() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Routes>
        <Route path="/cloud-accounts" element={<CloudAccounts />} />
        <Route path="/resources" element={<Resources />} />
        <Route path="/costs" element={<CostManagement />} />
      </Routes>
    </Suspense>
  );
}

// ✅ Correct - Component-level lazy loading for heavy components
const NetworkTopologyDiagram = lazy(() => 
  import('@/components/network-topology/NetworkTopologyDiagram')
);

function NetworkTab() {
  const [showDiagram, setShowDiagram] = useState(false);
  
  return (
    <div>
      <Button onClick={() => setShowDiagram(true)}>
        Load Network Diagram
      </Button>
      {showDiagram && (
        <Suspense fallback={<DiagramSkeleton />}>
          <NetworkTopologyDiagram />
        </Suspense>
      )}
    </div>
  );
}
```

#### 2. Memory Management
```typescript
// ✅ Correct - Cleanup and memory management
export function useResourceMetrics(resourceId: string) {
  return useQuery({
    queryKey: ['resource-metrics', resourceId],
    queryFn: () => resourcesApi.getMetrics(resourceId),
    enabled: !!resourceId,
    // Prevent memory leaks with shorter garbage collection
    gcTime: 2 * 60 * 1000, // 2 minutes
    // Reduce memory usage for large datasets
    select: (data) => ({
      cpu: data.metrics.cpu,
      memory: data.metrics.memory,
      // Only keep essential metrics
    }),
  });
}

// ✅ Correct - Cleanup subscriptions and intervals
function RealTimeMetrics({ resourceId }: { resourceId: string }) {
  useEffect(() => {
    const interval = setInterval(() => {
      queryClient.invalidateQueries(['resource-metrics', resourceId]);
    }, 30000);

    return () => clearInterval(interval); // Cleanup
  }, [resourceId]);
}
```

### Network Optimization

#### 1. Request Optimization
```typescript
// ✅ Correct - Pagination for large datasets
export function useResourcesPaginated(accountId: string, page = 1, limit = 20) {
  return useInfiniteQuery({
    queryKey: ['resources', accountId, 'paginated'],
    queryFn: ({ pageParam = 1 }) => 
      resourcesApi.getPaginated(accountId, pageParam, limit),
    getNextPageParam: (lastPage, pages) => {
      return lastPage.hasMore ? pages.length + 1 : undefined;
    },
    enabled: !!accountId,
  });
}

// ✅ Correct - Efficient filtering on client-side
function ResourcesList({ accountId }: { accountId: string }) {
  const { data: resources } = useResources(accountId);
  const [searchTerm, setSearchTerm] = useState('');
  
  // Client-side filtering to avoid unnecessary API calls
  const filteredResources = useMemo(() => {
    return resources?.filter(resource => 
      resource.name.toLowerCase().includes(searchTerm.toLowerCase())
    ) ?? [];
  }, [resources, searchTerm]);
  
  return (
    <div>
      <Input 
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search resources..."
      />
      <VirtualizedList items={filteredResources} />
    </div>
  );
}
```

## SEO Considerations

### Client-Side Rendering Limitations

**Important**: Since CostPie uses React/Vite (client-side rendering), SEO optimization is primarily focused on technical SEO and initial page load optimization rather than content indexing.

#### 1. Meta Tags Management
```typescript
// ✅ Correct - Dynamic meta tag management
import { Helmet } from 'react-helmet-async';

function CloudAccountsPage() {
  const { data: accounts } = useCloudAccounts();
  
  return (
    <>
      <Helmet>
        <title>Cloud Accounts - CostPie Dashboard</title>
        <meta 
          name="description" 
          content="Manage and monitor your cloud accounts. Track costs and optimize resources across AWS, Azure, and GCP." 
        />
        <meta name="robots" content="noindex, nofollow" /> {/* Private app */}
        <link rel="canonical" href={`${window.location.origin}/cloud-accounts`} />
      </Helmet>
      
      <div>
        <h1>Cloud Accounts ({accounts?.length || 0})</h1>
        {/* Component content */}
      </div>
    </>
  );
}
```

#### 2. Initial Loading Performance
```typescript
// ✅ Correct - Optimize initial page load
import { lazy, Suspense } from 'react';

// Critical components load immediately
import Header from '@/components/Header';
import Sidebar from '@/components/Sidebar';

// Non-critical components lazy load
const ResourcesList = lazy(() => import('@/components/resources/ResourcesList'));
const CostChart = lazy(() => import('@/components/dashboard/CostChart'));

function Dashboard() {
  return (
    <div className="app-layout">
      {/* Critical UI loads immediately */}
      <Header />
      <Sidebar />
      
      {/* Non-critical UI lazy loads */}
      <main>
        <Suspense fallback={<ResourcesSkeleton />}>
          <ResourcesList />
        </Suspense>
        
        <Suspense fallback={<ChartSkeleton />}>
          <CostChart />
        </Suspense>
      </main>
    </div>
  );
}
```

#### 3. Performance Metrics for SEO
```typescript
// ✅ Correct - Core Web Vitals optimization
function App() {
  useEffect(() => {
    // Track Core Web Vitals
    import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
      getCLS(console.log);
      getFID(console.log);
      getFCP(console.log);
      getLCP(console.log);
      getTTFB(console.log);
    });
  }, []);

  return (
    <ErrorBoundary>
      {/* Preload critical resources */}
      <link rel="preload" href="/api/organizations/mine" as="fetch" crossOrigin="anonymous" />
      <link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossOrigin="anonymous" />
      
      <Router>
        <Routes>
          {/* Your routes */}
        </Routes>
      </Router>
    </ErrorBoundary>
  );
}
```

### Technical SEO for SPA

#### 1. URL Structure & Routing
```typescript
// ✅ Correct - SEO-friendly URL structure
const router = createBrowserRouter([
  {
    path: "/",
    element: <Dashboard />,
  },
  {
    path: "/cloud-accounts",
    element: <CloudAccounts />,
    children: [
      { path: ":accountId", element: <CloudAccountDetails /> },
      { path: ":accountId/resources", element: <AccountResources /> },
    ],
  },
  {
    path: "/resources",
    element: <Resources />,
    children: [
      { path: ":resourceId", element: <ResourceDetails /> },
    ],
  },
  {
    path: "/costs",
    element: <CostManagement />,
    children: [
      { path: "analysis", element: <CostAnalysis /> },
      { path: "optimization", element: <CostOptimization /> },
    ],
  },
]);

// Clean, hierarchical URLs:
// /cloud-accounts/aws-prod-123
// /cloud-accounts/aws-prod-123/resources
// /resources/i-0abcd1234efgh5678
// /costs/analysis
```

#### 2. Error Handling for SEO
```typescript
// ✅ Correct - SEO-friendly error pages
function NotFoundPage() {
  return (
    <>
      <Helmet>
        <title>Page Not Found - CostPie</title>
        <meta name="robots" content="noindex, nofollow" />
      </Helmet>
      
      <div className="error-page">
        <h1>404 - Page Not Found</h1>
        <p>The page you're looking for doesn't exist.</p>
        <Link to="/" className="btn-primary">
          Return to Dashboard
        </Link>
      </div>
    </>
  );
}

function ErrorBoundary({ children }: { children: React.ReactNode }) {
  return (
    <ErrorBoundaryComponent
      FallbackComponent={({ error, resetError }) => (
        <>
          <Helmet>
            <title>Error - CostPie</title>
            <meta name="robots" content="noindex, nofollow" />
          </Helmet>
          
          <div className="error-boundary">
            <h1>Something went wrong</h1>
            <Button onClick={resetError}>Try again</Button>
          </div>
        </>
      )}
    >
      {children}
    </ErrorBoundaryComponent>
  );
}
```

## Type Safety Requirements

### Comprehensive TypeScript Standards

**CRITICAL**: All data fetching operations MUST be fully typed to prevent runtime errors and ensure API contract compliance.

#### 1. API Response Interfaces
```typescript
// ✅ Correct - Complete interface definitions
interface Organization {
  id: string;
  name: string;
  subscription_plan: 'free' | 'pro' | 'enterprise';
  created_at: string;
  updated_at: string;
  settings?: OrganizationSettings;
}

interface CloudAccount {
  id: string;
  organization_id: string;
  name: string;
  provider: 'aws' | 'azure' | 'gcp';
  region: string;
  status: 'active' | 'inactive' | 'error';
  credentials_encrypted: boolean;
  last_sync_at?: string;
  created_at: string;
  updated_at: string;
}

interface Resource {
  id: string;
  cloud_account_id: string;
  aws_resource_id: string;
  name: string;
  type: string;
  region: string;
  status: 'running' | 'stopped' | 'terminated';
  tags: Record<string, string>;
  cost_per_hour?: number;
  last_seen_at: string;
  created_at: string;
}

// ❌ Incorrect - Using 'any' type
interface BadResource {
  id: string;
  data: any; // Never use 'any'
  metadata: unknown; // Avoid 'unknown' without proper type guards
}
```

#### 2. API Request/Response Typing
```typescript
// ✅ Correct - Fully typed API functions
export const cloudAccountsApi = {
  async create(data: CreateCloudAccountDto): Promise<CloudAccount> {
    return apiClient.post<CloudAccount>('/cloud-accounts', data);
  },
  
  async getById(id: string): Promise<CloudAccount> {
    return apiClient.get<CloudAccount>(`/cloud-accounts/${id}`);
  },
  
  async update(id: string, data: Partial<UpdateCloudAccountDto>): Promise<CloudAccount> {
    return apiClient.put<CloudAccount>(`/cloud-accounts/${id}`, data);
  },
  
  async delete(id: string): Promise<void> {
    return apiClient.delete<void>(`/cloud-accounts/${id}`);
  },
  
  async getResources(id: string, filters?: ResourceFilters): Promise<Resource[]> {
    const params = filters ? new URLSearchParams(filters as any).toString() : '';
    return apiClient.get<Resource[]>(`/cloud-accounts/${id}/resources?${params}`);
  }
};

// ❌ Incorrect - Untyped API functions
export const badCloudAccountsApi = {
  async create(data: any): Promise<any> { // No type safety
    return apiClient.post('/cloud-accounts', data);
  }
};
```

#### 3. Form Data Types with Zod
```typescript
// ✅ Correct - Zod schema with TypeScript inference
import { z } from 'zod';

const createCloudAccountSchema = z.object({
  name: z.string()
    .min(1, 'Account name is required')
    .max(100, 'Account name must be less than 100 characters'),
  provider: z.enum(['aws', 'azure', 'gcp']),
  region: z.string().min(1, 'Region is required'),
  credentials: z.object({
    accessKeyId: z.string()
      .min(1, 'Access Key ID is required')
      .regex(/^AKIA[0-9A-Z]{16}$/, 'Invalid AWS Access Key ID format'),
    secretAccessKey: z.string()
      .min(40, 'Secret Access Key must be at least 40 characters'),
    sessionToken: z.string().optional(),
  }),
});

// Infer TypeScript type from Zod schema
type CreateCloudAccountDto = z.infer<typeof createCloudAccountSchema>;

// Use in forms
function CloudAccountForm() {
  const form = useForm<CreateCloudAccountDto>({
    resolver: zodResolver(createCloudAccountSchema),
    defaultValues: {
      name: '',
      provider: 'aws',
      region: 'us-east-1',
      credentials: {
        accessKeyId: '',
        secretAccessKey: '',
      },
    },
  });
  
  const onSubmit = async (data: CreateCloudAccountDto) => {
    // data is fully typed here
    await cloudAccountsApi.create(data);
  };
}
```

#### 4. Query Hook Type Safety
```typescript
// ✅ Correct - Typed query hooks
export function useCloudAccounts(organizationId: string) {
  return useQuery({
    queryKey: ['cloud-accounts', organizationId] as const,
    queryFn: (): Promise<CloudAccount[]> => 
      cloudAccountsApi.getByOrganization(organizationId),
    enabled: !!organizationId,
    select: (data: CloudAccount[]): CloudAccount[] => {
      // Type-safe data transformation
      return data.map(account => ({
        ...account,
        displayName: `${account.name} (${account.provider.toUpperCase()})`,
      }));
    },
  });
}

export function useCreateCloudAccount() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateCloudAccountDto): Promise<CloudAccount> => 
      cloudAccountsApi.create(data),
    onSuccess: (newAccount: CloudAccount) => {
      // Type-safe cache updates
      queryClient.setQueryData<CloudAccount[]>(
        ['cloud-accounts', newAccount.organization_id],
        (oldData) => oldData ? [...oldData, newAccount] : [newAccount]
      );
    },
    onError: (error: ApiError) => {
      // Type-safe error handling
      console.error('Failed to create cloud account:', error.message);
    },
  });
}

// ❌ Incorrect - Untyped hooks
export function badUseCloudAccounts(orgId: string) {
  return useQuery({
    queryKey: ['accounts'], // Not typed
    queryFn: () => fetch('/api/accounts').then(r => r.json()), // No types
    select: (data) => data.map(item => item.name), // No type safety
  });
}
```

#### 5. Error Type Safety
```typescript
// ✅ Correct - Typed error handling
interface ApiError extends Error {
  status: number;
  code: string;
  details?: Record<string, any>;
}

interface ValidationError extends ApiError {
  errors: Array<{
    field: string;
    message: string;
    code: string;
  }>;
}

const handleApiError = (error: unknown): string => {
  if (error instanceof Error) {
    const apiError = error as ApiError;
    
    switch (apiError.status) {
      case 400:
        return 'Invalid request. Please check your input.';
      case 401:
        return 'You are not authorized. Please log in again.';
      case 403:
        return 'You do not have permission to perform this action.';
      case 404:
        return 'The requested resource was not found.';
      case 422:
        const validationError = apiError as ValidationError;
        return validationError.errors
          ?.map(err => err.message)
          .join(', ') || 'Validation failed.';
      case 429:
        return 'Too many requests. Please try again later.';
      default:
        return 'An unexpected error occurred. Please try again.';
    }
  }
  
  return 'An unknown error occurred.';
};

// Usage in components
function MyComponent() {
  const mutation = useCreateCloudAccount();
  
  const handleSubmit = async (data: CreateCloudAccountDto) => {
    try {
      await mutation.mutateAsync(data);
    } catch (error) {
      const errorMessage = handleApiError(error);
      toast({
        title: 'Error',
        description: errorMessage,
        variant: 'destructive',
      });
    }
  };
}
```

#### 6. Strict TypeScript Configuration
```json
// ✅ Required tsconfig.json settings
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true
  }
}
```

## Consistency Patterns

### File Organization Standards

**CRITICAL**: Maintain consistent file organization and naming conventions across the entire application.

#### 1. Directory Structure Consistency
```
src/
├── components/           # Reusable UI components
│   ├── ui/              # shadcn/ui components
│   ├── dashboard/       # Dashboard-specific components
│   ├── cloud-ops/       # Cloud operations components
│   ├── cost-management/ # Cost management components
│   └── resources/       # Resource management components
├── hooks/               # Custom React hooks
│   └── queries/         # TanStack Query hooks
├── lib/                 # Utility libraries
│   ├── api/            # API client functions
│   └── utils.ts        # General utilities
├── pages/              # Page-level components
└── types/              # TypeScript type definitions
```

#### 2. Naming Conventions
```typescript
// ✅ Correct - Consistent naming patterns
// Files: kebab-case
cloud-accounts.ts
cost-management.tsx
network-topology.tsx

// Components: PascalCase
function CloudAccountsList() {}
function CostManagementDashboard() {}
function NetworkTopologyDiagram() {}

// API functions: camelCase with descriptive names
export const cloudAccountsApi = {
  getByOrganization,
  create,
  update,
  delete,
  syncResources,
};

// Hooks: use + PascalCase
useCloudAccounts()
useCreateCloudAccount()
useOptimisticResourceUpdate()

// Types/Interfaces: PascalCase
interface CloudAccount {}
type CreateCloudAccountDto = {};
enum ResourceStatus {}

// Constants: SCREAMING_SNAKE_CASE
const API_BASE_URL = 'https://api.costpie.com';
const MAX_RETRY_ATTEMPTS = 3;
const CACHE_STALE_TIME = 5 * 60 * 1000;
```

### API Layer Consistency

#### 1. Standardized API Client Pattern
```typescript
// ✅ Correct - Consistent API client structure for all modules
export const resourcesApi = {
  // READ operations
  async getByAccount(accountId: string, filters?: ResourceFilters): Promise<Resource[]> {
    const params = filters ? new URLSearchParams(filters as any).toString() : '';
    return apiClient.get<Resource[]>(`/cloud-accounts/${accountId}/resources?${params}`);
  },
  
  async getById(id: string): Promise<Resource> {
    return apiClient.get<Resource>(`/resources/${id}`);
  },
  
  async getMetrics(id: string): Promise<ResourceMetrics> {
    return apiClient.get<ResourceMetrics>(`/resources/${id}/metrics`);
  },
  
  // WRITE operations
  async update(id: string, data: Partial<UpdateResourceDto>): Promise<Resource> {
    return apiClient.put<Resource>(`/resources/${id}`, data);
  },
  
  async assignToTeam(resourceId: string, teamId: string): Promise<void> {
    return apiClient.post<void>(`/resources/${resourceId}/team/${teamId}`);
  },
  
  async removeFromTeam(resourceId: string): Promise<void> {
    return apiClient.delete<void>(`/resources/${resourceId}/team`);
  },
  
  // ACTIONS
  async start(id: string): Promise<void> {
    return apiClient.post<void>(`/resources/${id}/start`);
  },
  
  async stop(id: string): Promise<void> {
    return apiClient.post<void>(`/resources/${id}/stop`);
  },
  
  // BATCH operations
  async batchUpdate(operations: ResourceOperation[]): Promise<Resource[]> {
    return apiClient.post<Resource[]>('/resources/batch', { operations });
  },
};

// Follow same pattern for all API modules:
// organizationsApi, cloudAccountsApi, costsApi, tagsApi, etc.
```

#### 2. Consistent Query Hook Patterns
```typescript
// ✅ Correct - Standardized hook patterns for all entities
// 1. Base query hook
export function useResources(accountId: string, filters?: ResourceFilters) {
  return useQuery({
    queryKey: ['resources', accountId, filters] as const,
    queryFn: () => resourcesApi.getByAccount(accountId, filters),
    enabled: !!accountId,
    staleTime: 2 * 60 * 1000,
  });
}

// 2. Individual item hook
export function useResource(id: string) {
  return useQuery({
    queryKey: ['resources', id] as const,
    queryFn: () => resourcesApi.getById(id),
    enabled: !!id,
    staleTime: 2 * 60 * 1000,
  });
}

// 3. Create mutation hook
export function useCreateResource() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: resourcesApi.create,
    onSuccess: (newResource) => {
      queryClient.setQueryData<Resource[]>(
        ['resources', newResource.cloud_account_id],
        (oldData) => oldData ? [...oldData, newResource] : [newResource]
      );
    },
  });
}

// 4. Update mutation hook
export function useUpdateResource(id: string) {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (data: Partial<UpdateResourceDto>) => 
      resourcesApi.update(id, data),
    onSuccess: (updatedResource) => {
      queryClient.setQueryData(['resources', id], updatedResource);
      queryClient.invalidateQueries(['resources']);
    },
  });
}

// 5. Delete mutation hook
export function useDeleteResource() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: resourcesApi.delete,
    onSuccess: (_, deletedId) => {
      queryClient.removeQueries(['resources', deletedId]);
      queryClient.invalidateQueries(['resources']);
    },
  });
}

// Apply this exact pattern to all entities:
// useOrganizations, useCloudAccounts, useCosts, etc.
```

### Error Handling Consistency

#### 1. Standardized Error Response Format
```typescript
// ✅ Correct - Consistent error handling across all components
interface StandardErrorResponse {
  error: string;
  message: string;
  statusCode: number;
  timestamp: string;
  path?: string;
  errors?: ValidationError[];
}

// Standard error display component
function ErrorAlert({ error, onRetry }: { error: ApiError; onRetry?: () => void }) {
  const message = getErrorMessage(error);
  
  return (
    <Alert variant="destructive">
      <AlertCircle className="h-4 w-4" />
      <AlertTitle>Error</AlertTitle>
      <AlertDescription className="mt-2">
        {message}
        {onRetry && (
          <Button variant="outline" size="sm" onClick={onRetry} className="mt-2">
            Try Again
          </Button>
        )}
      </AlertDescription>
    </Alert>
  );
}

// Use in all components consistently
function ResourcesList() {
  const { data: resources, error, isLoading, refetch } = useResources(accountId);
  
  if (error) {
    return <ErrorAlert error={error} onRetry={() => refetch()} />;
  }
  
  if (isLoading) {
    return <ResourcesSkeleton />;
  }
  
  return <ResourcesTable resources={resources} />;
}
```

#### 2. Consistent Loading States
```typescript
// ✅ Correct - Standardized loading components
function ResourcesSkeleton() {
  return (
    <div className="space-y-4">
      {Array.from({ length: 5 }).map((_, i) => (
        <div key={i} className="flex items-center space-x-4">
          <Skeleton className="h-12 w-12 rounded-full" />
          <div className="space-y-2">
            <Skeleton className="h-4 w-[250px]" />
            <Skeleton className="h-4 w-[200px]" />
          </div>
        </div>
      ))}
    </div>
  );
}

// Use consistent skeleton patterns across all list components:
// CloudAccountsSkeleton, CostChartSkeleton, TeamsSkeleton, etc.
```

### Form Handling Consistency

#### 1. Standardized Form Component Pattern
```typescript
// ✅ Correct - Consistent form structure for all entities
function CloudAccountForm({ 
  initialData, 
  onSuccess, 
  onCancel 
}: {
  initialData?: Partial<CloudAccount>;
  onSuccess: (account: CloudAccount) => void;
  onCancel: () => void;
}) {
  const isEditing = !!initialData?.id;
  const createMutation = useCreateCloudAccount();
  const updateMutation = useUpdateCloudAccount(initialData?.id!);
  
  const form = useForm<CreateCloudAccountDto>({
    resolver: zodResolver(createCloudAccountSchema),
    defaultValues: {
      name: initialData?.name || '',
      provider: initialData?.provider || 'aws',
      region: initialData?.region || 'us-east-1',
      credentials: {
        accessKeyId: '',
        secretAccessKey: '',
      },
    },
  });

  const onSubmit = async (data: CreateCloudAccountDto) => {
    try {
      const result = isEditing 
        ? await updateMutation.mutateAsync(data)
        : await createMutation.mutateAsync(data);
      
      toast({
        title: 'Success',
        description: `Cloud account ${isEditing ? 'updated' : 'created'} successfully`,
      });
      
      onSuccess(result);
    } catch (error) {
      toast({
        title: 'Error',
        description: `Failed to ${isEditing ? 'update' : 'create'} cloud account`,
        variant: 'destructive',
      });
    }
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
        {/* Form fields */}
        
        <div className="flex justify-end space-x-2">
          <Button type="button" variant="outline" onClick={onCancel}>
            Cancel
          </Button>
          <Button 
            type="submit" 
            disabled={createMutation.isPending || updateMutation.isPending}
          >
            {isEditing ? 'Update' : 'Create'} Account
          </Button>
        </div>
      </form>
    </Form>
  );
}

// Apply this pattern to all form components:
// OrganizationForm, ResourceForm, TeamForm, etc.
```

### Best Practices Enforcement

#### 1. Code Review Checklist
- ✅ All API calls are typed with interfaces
- ✅ TanStack Query hooks follow naming conventions
- ✅ Error handling uses standardized components
- ✅ Loading states use consistent skeleton components
- ✅ Forms use React Hook Form + Zod validation
- ✅ Components follow single responsibility principle
- ✅ File names follow kebab-case convention
- ✅ No direct fetch/axios calls in components
- ✅ Proper cache invalidation strategies implemented

#### 2. Development Workflow Standards
```typescript
// ✅ Correct - Development checklist for new features
// 1. Create TypeScript interfaces first
// 2. Implement API client functions with full typing
// 3. Create TanStack Query hooks following patterns
// 4. Build components with error boundaries
// 5. Add form validation with Zod schemas
// 6. Implement loading states and skeletons
// 7. Add comprehensive error handling
// 8. Test all paths (success, error, loading)
// 9. Update documentation if needed
// 10. Review code against consistency checklist
```

This comprehensive update ensures CostPie's data fetching architecture maintains security, performance, SEO optimization, type safety, and consistency across the entire application.