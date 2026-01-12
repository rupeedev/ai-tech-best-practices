# Frontend Data Mutations

This document outlines the coding standards and patterns for data mutations in the CostPie React/Vite frontend application.

## Architecture Overview

Frontend data mutations follow a client-side architecture pattern:

1. **React Components** - Handle UI state, forms, and user interactions
2. **API Client Layer** (`src/lib/api/`) - HTTP communication with backend
3. **TanStack Query** - Server state management, caching, and mutation handling
4. **Backend NestJS API** - External HTTP API endpoints

## Frontend Data Flow Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│ React Component │───▶│   API Client     │───▶│  HTTP Request   │
│   (Forms/UI)    │    │ (api-client.ts)  │    │  (Backend API)  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       ▼
         │                       │              ┌─────────────────┐
         │                       │              │   NestJS API    │
         │                       │              │   (Backend)     │
         │                       │              └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  TanStack Query │    │   Error Handler  │    │    Database     │
│   (Caching)     │    │  (Error Logging) │    │   (Prisma)      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │    Optimistic Updates    │
                    │   Cache Invalidation     │
                    └──────────────────────────┘
```

## Technology Stack

- **Framework**: React 18 with Vite
- **State Management**: TanStack Query for server state
- **Form Handling**: React Hook Form with Zod validation
- **HTTP Client**: Custom API client with automatic retry and auth
- **UI Components**: shadcn/ui with Tailwind CSS
- **Error Handling**: Custom error boundary and logging system

## Frontend MUST Follow React/Vite Best Practices

### ✅ Current Architecture Standards:
- **React components for UI** - Functional components with hooks
- **TanStack Query for server state** - No manual state management for server data
- **API client for HTTP communication** - Centralized HTTP layer
- **React Hook Form for form handling** - Type-safe form management
- **Client-side validation and error handling** - Comprehensive error boundaries

### Current Frontend Architecture Pattern:
```
React Component → API Client → HTTP Request → Backend NestJS → Database
```

## API Client Layer

### File Organization

API client functions MUST be organized in the `src/lib/api/` directory by feature:

```typescript
// Example file structure
src/lib/api/
├── api-client.ts          // Base HTTP client
├── organizations.ts       // Organization-related API calls
├── cloud-accounts.ts      // Cloud account operations
├── resources.ts           // Resource management
├── costs.ts              // Cost data APIs
├── tags.ts               // Tagging operations
├── users.ts              // User management
├── teams.ts              // Team operations
└── types.ts              // Shared TypeScript interfaces
```

### API Client Standards

- **Typed Interfaces**: All API functions MUST use TypeScript interfaces
- **Error Handling**: Use the shared error handling utilities
- **Authentication**: Automatic JWT token injection via api-client
- **Retry Logic**: Configurable retry with exponential backoff

```typescript
// ✅ Correct - Typed API function
interface CreateOrganizationDto {
  name: string;
  subscription_plan?: string;
}

export const organizationsApi = {
  async create(data: CreateOrganizationDto): Promise<Organization> {
    return apiClient.post<Organization>('/organizations', data);
  },
  
  async getAll(): Promise<Organization[]> {
    return apiClient.get<Organization[]>('/organizations/mine');
  },
  
  async update(id: string, data: Partial<CreateOrganizationDto>): Promise<Organization> {
    return apiClient.put<Organization>(`/organizations/${id}`, data);
  }
};

// ❌ Incorrect - Untyped API function
export function createOrg(data: any) {
  return fetch('/api/organizations', { method: 'POST', body: JSON.stringify(data) });
}
```

### Base API Client Configuration

The base API client (`api-client.ts`) handles:

```typescript
// Core features of api-client.ts
const apiClient = {
  // Automatic JWT token injection
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json',
    'x-organization-id': organizationId
  },
  
  // Automatic retry with exponential backoff
  retry: {
    maxRetries: 3,
    retryDelay: 1000,
    retryStatuses: [408, 429, 500, 502, 503, 504]
  },
  
  // Error handling and logging
  errorHandler: (error) => {
    errorLogger.logError(error, { context: 'apiClient' });
    throw error;
  }
};
```

## TanStack Query Hooks

### File Organization

Query and mutation hooks MUST be placed in `src/hooks/queries/` directory:

```typescript
// Example file structure
src/hooks/queries/
├── index.ts                    // Re-export all hooks
├── use-organizations.ts        // Organization queries/mutations
├── use-cloud-accounts.ts       // Cloud account operations
├── use-resources.ts           // Resource management
├── use-costs.ts               // Cost data queries
└── use-current-month-costs.ts // Specific cost queries
```

### Query Hook Patterns

- **Queries**: Use `useQuery` for data fetching
- **Mutations**: Use `useMutation` for data modifications
- **Cache Management**: Implement proper invalidation strategies
- **Error Handling**: Use built-in TanStack Query error handling

```typescript
// ✅ Correct - Proper query hook implementation
export function useOrganizations() {
  return useQuery({
    queryKey: ['organizations'],
    queryFn: organizationsApi.getAll,
    staleTime: 5 * 60 * 1000, // 5 minutes
    retry: (failureCount, error) => {
      return isRetryableError(error) && failureCount < 3;
    }
  });
}

// ✅ Correct - Proper mutation hook implementation
export function useCreateOrganization() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateOrganizationDto) => organizationsApi.create(data),
    onSuccess: () => {
      // Invalidate and refetch organizations list
      queryClient.invalidateQueries({ queryKey: ['organizations'] });
    },
    onError: (error) => {
      // Error is automatically handled by TanStack Query
      console.error('Failed to create organization:', error);
    }
  });
}

// ❌ Incorrect - Direct API calls in components
function MyComponent() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    organizationsApi.getAll().then(setData); // Don't do this
  }, []);
}
```

### Cache Invalidation Strategies

Implement proper cache invalidation to maintain data consistency:

```typescript
// Strategy 1: Invalidate related queries
export function useUpdateOrganization(id: string) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: Partial<CreateOrganizationDto>) => 
      organizationsApi.update(id, data),
    onSuccess: (updatedOrg) => {
      // Update specific item in cache
      queryClient.setQueryData(['organizations', id], updatedOrg);
      
      // Invalidate list to trigger refetch
      queryClient.invalidateQueries({ queryKey: ['organizations'] });
    },
  });
}

// Strategy 2: Optimistic updates for better UX
export function useDeleteOrganization() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: string) => organizationsApi.delete(id),
    onMutate: async (deletedId) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['organizations'] });
      
      // Snapshot the previous value
      const previousOrgs = queryClient.getQueryData(['organizations']);
      
      // Optimistically remove the organization
      queryClient.setQueryData(['organizations'], (old: Organization[]) =>
        old?.filter(org => org.id !== deletedId) ?? []
      );
      
      return { previousOrgs };
    },
    onError: (err, deletedId, context) => {
      // Rollback on error
      queryClient.setQueryData(['organizations'], context?.previousOrgs);
    },
    onSettled: () => {
      // Always refetch after mutation
      queryClient.invalidateQueries({ queryKey: ['organizations'] });
    },
  });
}
```

## Form Handling with React Hook Form

### Form Component Patterns

Use React Hook Form with Zod validation for type-safe form handling:

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// ✅ Correct - Zod schema validation
const organizationSchema = z.object({
  name: z.string().min(1, 'Organization name is required'),
  subscription_plan: z.string().optional(),
});

type OrganizationFormData = z.infer<typeof organizationSchema>;

function OrganizationForm() {
  const createMutation = useCreateOrganization();
  
  const form = useForm<OrganizationFormData>({
    resolver: zodResolver(organizationSchema),
    defaultValues: {
      name: '',
      subscription_plan: 'free',
    },
  });

  const onSubmit = async (data: OrganizationFormData) => {
    try {
      await createMutation.mutateAsync(data);
      toast({
        title: 'Success',
        description: 'Organization created successfully',
      });
      // Navigation handled after successful mutation
      navigate('/dashboard');
    } catch (error) {
      // Error already handled by mutation hook
      toast({
        title: 'Error',
        description: 'Failed to create organization',
        variant: 'destructive',
      });
    }
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)}>
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Organization Name</FormLabel>
              <FormControl>
                <Input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        
        <Button 
          type="submit" 
          disabled={createMutation.isPending}
        >
          {createMutation.isPending ? 'Creating...' : 'Create Organization'}
        </Button>
      </form>
    </Form>
  );
}
```

### Form Validation Standards

- **Client-side Validation**: Use Zod schemas for type safety
- **Real-time Validation**: React Hook Form handles validation on blur/change
- **Server-side Validation**: Backend validation errors displayed via form state
- **Error Display**: Use FormMessage components for consistent error styling

```typescript
// ✅ Correct - Comprehensive validation
const cloudAccountSchema = z.object({
  name: z.string()
    .min(1, 'Account name is required')
    .max(100, 'Account name must be less than 100 characters'),
  region: z.string()
    .min(1, 'Region is required'),
  credentials: z.object({
    accessKeyId: z.string()
      .min(1, 'Access Key ID is required')
      .regex(/^AKIA[0-9A-Z]{16}$/, 'Invalid Access Key ID format'),
    secretAccessKey: z.string()
      .min(1, 'Secret Access Key is required')
      .min(40, 'Secret Access Key must be at least 40 characters'),
  }),
});

// ❌ Incorrect - No validation
function BadForm() {
  const [name, setName] = useState('');
  
  const handleSubmit = () => {
    // No validation - data could be invalid
    createMutation.mutate({ name });
  };
}
```

## Error Handling

### Error Boundary Implementation

Use React Error Boundaries to catch and handle rendering errors:

```typescript
// ✅ Correct - Error boundary usage
function App() {
  return (
    <ErrorBoundary
      fallback={<ErrorFallback />}
      onError={(error, errorInfo) => {
        errorLogger.logError(error, { 
          context: 'React Error Boundary',
          errorInfo 
        });
      }}
    >
      <Router>
        <Routes>
          {/* Your routes */}
        </Routes>
      </Router>
    </ErrorBoundary>
  );
}

// Error fallback component
function ErrorFallback({ error, resetError }) {
  return (
    <div className="error-boundary">
      <h2>Something went wrong</h2>
      <p>{error.message}</p>
      <Button onClick={resetError}>Try again</Button>
    </div>
  );
}
```

### API Error Handling

Handle API errors consistently across the application:

```typescript
// Built into api-client.ts
function handleApiError(response: Response, data: any): never {
  const errorObj = new Error(
    Array.isArray(data.message) 
      ? data.message.join(', ') 
      : data.message || `API request failed with status ${response.status}`
  );
  
  errorObj.status = response.status;
  errorObj.code = data.error;
  errorObj.details = data.details;
  
  // Log error for monitoring
  errorLogger.logError(errorObj, {
    context: 'API Error',
    url: response.url,
    status: response.status,
  });
  
  throw errorObj;
}

// Usage in components
function MyComponent() {
  const query = useOrganizations();
  
  if (query.error) {
    return (
      <Alert variant="destructive">
        <AlertTitle>Error</AlertTitle>
        <AlertDescription>
          {query.error.message || 'Failed to load organizations'}
        </AlertDescription>
      </Alert>
    );
  }
  
  // Render success state
}
```

## Optimistic Updates

### Custom Hook for Optimistic Mutations

Use the provided `useOptimisticMutation` hook for better UX:

```typescript
// ✅ Correct - Optimistic updates with rollback
function useUpdateCloudAccount(accountId: string) {
  return useOptimisticMutation({
    mutationFn: (data: UpdateCloudAccountDto) => 
      cloudAccountsApi.update(accountId, data),
    queryKey: ['cloud-accounts', accountId],
    getOptimisticData: (variables, currentData) => ({
      ...currentData,
      ...variables,
      // Optimistically update the data
    }),
    invalidateQueries: (data) => {
      // Invalidate related queries after success
      queryClient.invalidateQueries(['cloud-accounts']);
    },
    toasts: {
      optimistic: { 
        title: 'Updating...', 
        description: 'Saving your changes' 
      },
      success: { 
        title: 'Updated', 
        description: 'Account updated successfully' 
      },
      error: { 
        title: 'Error', 
        description: 'Failed to update account' 
      },
    },
  });
}

// Usage in component
function AccountEditor() {
  const updateMutation = useUpdateCloudAccount(accountId);
  
  const handleSave = (data) => {
    updateMutation.mutate(data);
    // UI immediately shows optimistic state
    // Rollback happens automatically on error
  };
}
```

## State Management Patterns

### Server State vs Client State

Clearly separate server state (managed by TanStack Query) from client state:

```typescript
// ✅ Correct - Server state with TanStack Query
function ResourcesList() {
  const { data: resources, isLoading } = useResources(accountId);
  
  // Client state for UI interactions
  const [selectedResources, setSelectedResources] = useState<string[]>([]);
  const [searchTerm, setSearchTerm] = useState('');
  
  // Derived state for filtering
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
      {/* Render filtered resources */}
    </div>
  );
}

// ❌ Incorrect - Managing server data in component state
function BadResourcesList() {
  const [resources, setResources] = useState([]);
  
  useEffect(() => {
    // Don't manage server state manually
    fetchResources().then(setResources);
  }, []);
}
```

## Performance Optimization

### Query Optimization

Implement proper query optimization strategies:

```typescript
// ✅ Correct - Optimized queries
export function useResources(accountId: string, filters?: ResourceFilters) {
  return useQuery({
    queryKey: ['resources', accountId, filters],
    queryFn: () => resourcesApi.getByAccount(accountId, filters),
    enabled: !!accountId, // Only run if accountId exists
    staleTime: 2 * 60 * 1000, // 2 minutes
    select: (data) => {
      // Transform data if needed
      return data.map(resource => ({
        ...resource,
        displayName: `${resource.name} (${resource.type})`,
      }));
    },
  });
}
```

### Bundle Optimization

Follow code splitting and lazy loading patterns:

```typescript
// ✅ Correct - Route-based code splitting
import { lazy, Suspense } from 'react';

const CloudAccounts = lazy(() => import('@/pages/CloudAccounts'));
const Resources = lazy(() => import('@/pages/Resources'));
const CostManagement = lazy(() => import('@/pages/CostManagement'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/cloud-accounts" element={<CloudAccounts />} />
        <Route path="/resources" element={<Resources />} />
        <Route path="/costs" element={<CostManagement />} />
      </Routes>
    </Suspense>
  );
}
```

## Best Practices Summary

### ✅ DO:
1. **Use TanStack Query** for all server state management
2. **Type everything** with TypeScript interfaces
3. **Validate forms** with Zod and React Hook Form
4. **Handle errors gracefully** with error boundaries and proper error states
5. **Implement optimistic updates** for better user experience
6. **Cache invalidation** strategies for data consistency
7. **Code splitting** for performance optimization
8. **Test components and API calls** separately

### ❌ DON'T:
1. **Manage server data in component state** - use TanStack Query
2. **Make direct API calls in components** - use custom hooks
3. **Skip validation** - always validate user input
4. **Ignore error handling** - implement comprehensive error boundaries
5. **Forget cache invalidation** - maintain data consistency
6. **Bundle everything together** - implement code splitting
7. **Skip testing** - test both components and API interactions

## Migration from Other Patterns

If migrating from other state management patterns:

### From Redux/Zustand
```typescript
// ❌ Old Redux pattern
const slice = createSlice({
  name: 'organizations',
  initialState: { data: [], loading: false },
  reducers: {
    fetchStart: (state) => { state.loading = true; },
    fetchSuccess: (state, action) => { 
      state.data = action.payload; 
      state.loading = false;
    },
  },
});

// ✅ New TanStack Query pattern
export function useOrganizations() {
  return useQuery({
    queryKey: ['organizations'],
    queryFn: organizationsApi.getAll,
  });
}
```

### From useState + useEffect
```typescript
// ❌ Old manual pattern
function OrganizationsList() {
  const [orgs, setOrgs] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    setLoading(true);
    organizationsApi.getAll()
      .then(setOrgs)
      .finally(() => setLoading(false));
  }, []);
}

// ✅ New TanStack Query pattern
function OrganizationsList() {
  const { data: orgs, isLoading } = useOrganizations();
}
```

This frontend data mutation pattern ensures type safety, proper error handling, optimal performance, and excellent user experience while maintaining clean separation of concerns between UI and data management layers.