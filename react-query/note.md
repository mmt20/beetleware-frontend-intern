# React Query Best Practices - Complete Guide

## Performance & Caching Strategy

### Query Keys
- **Use hierarchical keys**: `['users', userId, 'posts']`
- **Include all variables**: Keys must reflect data dependencies
- **Leverage partial matching**: `queryClient.invalidateQueries(['users'])` invalidates all user queries
- **Serializable values only**: Avoid functions/objects, use primitives
- **Consistent ordering**: `['users', id, 'posts']` not `['posts', id, 'users']`

### Stale Time Configuration
```tsx
useQuery(['user', id], fetchUser, {
  staleTime: 5 * 60 * 1000, // 5 minutes
})
```
- **Default (0ms)**: Refetch on every mount/focus
- **5-30min**: Good for stable data (user profiles, settings)
- **1-5min**: Semi-dynamic data (dashboards, analytics)
- **Infinity**: Static data (country lists, constants)
- **Context-aware**: Higher during user activity, lower during idle

### Cache Time vs Stale Time
- **staleTime**: When data becomes "old" (triggers background refetch)
- **cacheTime**: How long unused data stays in memory (default: 5min)
- **Pattern**: `staleTime < cacheTime` ensures cached data available during background refresh
- **Unused data**: Only removed after cacheTime expires with no observers
- **Memory management**: Lower cacheTime for large datasets

### Prefetching
```tsx
// On hover/route anticipation
queryClient.prefetchQuery(['post', id], fetchPost)

// Preload next page
queryClient.prefetchQuery(['posts', page + 1], () => fetchPosts(page + 1))

// Prefetch on idle
useEffect(() => {
  const idleCallback = requestIdleCallback(() => {
    queryClient.prefetchQuery(['heavyData'], fetchHeavyData)
  })
  return () => cancelIdleCallback(idleCallback)
}, [])
```
- **Eliminates loading states** for predicted navigation
- **Preload critical paths**: Next page, likely clicked items
- **Use staleTime with prefetch**: Prevent immediate refetch after navigation
- **Idle prefetching**: Load non-critical data during browser idle time

### Initial Data Strategies
```tsx
// From other query
useQuery(['user', id], fetchUser, {
  initialData: () => 
    queryClient.getQueryData(['users'])?.find(u => u.id === id),
  initialDataUpdatedAt: () => 
    queryClient.getQueryState(['users'])?.dataUpdatedAt
})

// From props/route state
useQuery(['post', id], fetchPost, {
  initialData: location.state?.post,
  staleTime: location.state?.post ? 0 : undefined // Immediately revalidate
})

// Placeholder data (doesn't persist)
useQuery(['posts'], fetchPosts, {
  placeholderData: { posts: [], total: 0 } // Shown while loading, not cached
})
```

## Rerender Control

### Select Option
```tsx
useQuery(['users'], fetchUsers, {
  select: useCallback((data) => data.map(u => u.name), [])
})

// Nested selection
useQuery(['org'], fetchOrg, {
  select: (data) => data.teams.find(t => t.id === teamId)
})

// Multiple selectors
const userNames = useQuery(['users'], fetchUsers, {
  select: (data) => data.map(u => u.name)
})
const userIds = useQuery(['users'], fetchUsers, {
  select: (data) => data.map(u => u.id)
})
// Same network request, different derived state
```
- **Memoizes transformations**: Prevents recompute on unrelated updates
- **Structural sharing**: React Query compares old/new, only rerenders on actual changes
- **Stabilize selector**: Use `useCallback` if selector uses external deps
- **Multiple components**: Can use different selectors on same query

### Disabled Queries
```tsx
useQuery(['user', id], fetchUser, {
  enabled: !!id // Don't run until id exists
})

// Conditional chains
const { data: user } = useQuery(['user'], fetchUser)
const { data: permissions } = useQuery(
  ['permissions', user?.role],
  () => fetchPermissions(user.role),
  { enabled: !!user?.role }
)
```
- **Prevents unnecessary requests** when dependencies missing
- **Stops polling**: `enabled: false` pauses refetch intervals
- **Manual triggering**: Use `refetch()` from disabled query
- **Dependent queries**: Chain with enabled option

### NotifyOnChangeProps
```tsx
useQuery(['data'], fetch, {
  notifyOnChangeProps: ['data'] // Only rerender on data change
})

// Tracked queries (auto-detect usage)
useQuery(['data'], fetch, {
  notifyOnChangeProps: 'tracked' // Only rerender for accessed properties
})
```
- **Granular control**: Specify which properties trigger rerenders
- **Tracked mode**: Automatically tracks accessed properties
- **Performance**: Prevents rerenders from `isLoading`, `isFetching` changes

### Query Observers
```tsx
// Subscribe without triggering rerenders
useEffect(() => {
  const unsubscribe = queryClient.getQueryCache().subscribe((event) => {
    if (event.type === 'updated' && event.query.queryKey[0] === 'notifications') {
      // Handle update without rerendering component
      showToast('New notification')
    }
  })
  return unsubscribe
}, [])
```

## Network Optimization

### Request Deduplication
- **Automatic**: Multiple components requesting same key = 1 network call
- **Window**: Default deduplication within render cycle
- **Cross-component**: Works even in different React trees
- **Manual prevention**: Use `queryClient.fetchQuery` to bypass deduplication

### Retry Strategy
```tsx
useQuery(['data'], fetch, {
  retry: 3,
  retryDelay: attemptIndex => Math.min(1000 * 2 ** attemptIndex, 30000)
})

// Conditional retry
useQuery(['data'], fetch, {
  retry: (failureCount, error) => {
    if (error.status === 404) return false
    if (error.status >= 500) return failureCount < 3
    return false
  }
})
```
- **Exponential backoff**: Prevents server hammering
- **Disable for 4xx**: Client errors unlikely to resolve on retry
- **Jitter**: Add randomness to prevent thundering herd
- **Network errors**: Always retry network failures

### Parallel Queries
```tsx
// BAD: Sequential waterfalls
const { data: user } = useQuery(['user'], fetchUser)
const { data: posts } = useQuery(['posts'], fetchPosts, { enabled: !!user })

// GOOD: Parallel when possible
useQueries([
  { queryKey: ['user'], queryFn: fetchUser },
  { queryKey: ['posts'], queryFn: fetchPosts }
])

// With dynamic keys
const postQueries = useQueries(
  postIds.map(id => ({
    queryKey: ['post', id],
    queryFn: () => fetchPost(id),
    staleTime: Infinity
  }))
)
```

### Pagination & Infinite Queries
```tsx
// Infinite scroll
useInfiniteQuery(['posts'], fetchPosts, {
  getNextPageParam: (lastPage) => lastPage.nextCursor,
  getPreviousPageParam: (firstPage) => firstPage.prevCursor,
  // Keeps all pages in cache
})

// Standard pagination with placeholderData
useQuery(['posts', page], () => fetchPosts(page), {
  placeholderData: keepPreviousData, // No loading state between pages
  staleTime: 5 * 60 * 1000
})

// Bi-directional infinite
const {
  data,
  fetchNextPage,
  fetchPreviousPage,
  hasNextPage,
  hasPreviousPage
} = useInfiniteQuery(['messages'], fetchMessages, {
  getNextPageParam: (lastPage) => lastPage.nextCursor,
  getPreviousPageParam: (firstPage) => firstPage.prevCursor
})
```

### Request Cancellation
```tsx
const fetchUser = ({ signal }) => {
  return fetch(`/api/users/${id}`, { signal })
}

useQuery(['user', id], fetchUser)
// Automatically cancelled when component unmounts or key changes
```

### Batching Requests
```tsx
// Custom batching with DataLoader pattern
const batchFn = async (ids) => {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ ids })
  })
  return response.json()
}

// Use with useQueries
const userQueries = useQueries(
  userIds.map(id => ({
    queryKey: ['user', id],
    queryFn: () => batchFn([id]) // Implement proper batching logic
  }))
)
```

## Avoiding Unnecessary Work

### Query Function Stability
```tsx
// BAD: New function every render
useQuery(['user'], () => api.get(`/users/${id}`))

// GOOD: Stable reference
const fetchUser = useCallback(() => api.get(`/users/${id}`), [id])
useQuery(['user', id], fetchUser)

// BEST: Plain function outside component
const fetchUser = (id) => api.get(`/users/${id}`)
useQuery(['user', id], () => fetchUser(id))

// With query key factory
const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters) => [...userKeys.lists(), { filters }] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id) => [...userKeys.details(), id] as const,
}
```

### Selective Subscriptions
```tsx
// Only rerender when specific field changes
const userName = useQuery(['user'], fetchUser, {
  select: (data) => data.name,
})

// Combine multiple fields efficiently
const userInfo = useQuery(['user'], fetchUser, {
  select: useCallback((data) => ({
    name: data.name,
    email: data.email
  }), [])
})
```

### Background Refetch Control
```tsx
{
  refetchOnWindowFocus: false, // Disable if data rarely changes
  refetchOnMount: false,       // Use cached data if fresh
  refetchInterval: false,      // Disable polling for static data
  refetchIntervalInBackground: false, // Stop polling when tab hidden
  refetchOnReconnect: true,    // Refetch when connection restored
}

// Dynamic refetch interval
useQuery(['live-data'], fetchData, {
  refetchInterval: (data) => data?.shouldPoll ? 1000 : false
})

// Conditional focus refetch
useQuery(['data'], fetchData, {
  refetchOnWindowFocus: (query) => 
    Date.now() - query.state.dataUpdatedAt > 30000 // Only if > 30s old
})
```

### Mutation Optimizations
```tsx
useMutation(updateUser, {
  onMutate: async (newUser) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries(['user', id])
    
    // Snapshot previous value
    const previous = queryClient.getQueryData(['user', id])
    
    // Optimistic update
    queryClient.setQueryData(['user', id], (old) => ({
      ...old,
      ...newUser
    }))
    
    return { previous } // Rollback context
  },
  onError: (err, variables, context) => {
    // Rollback on error
    queryClient.setQueryData(['user', id], context.previous)
  },
  onSettled: () => {
    // Revalidate after mutation
    queryClient.invalidateQueries(['user', id])
  }
})

// Optimistic list update
useMutation(addTodo, {
  onMutate: async (newTodo) => {
    await queryClient.cancelQueries(['todos'])
    const previous = queryClient.getQueryData(['todos'])
    
    queryClient.setQueryData(['todos'], (old) => [...old, {
      ...newTodo,
      id: Date.now(), // Temporary ID
      optimistic: true
    }])
    
    return { previous }
  },
  onSuccess: (savedTodo, variables, context) => {
    // Replace temp item with real one
    queryClient.setQueryData(['todos'], (old) =>
      old.map(todo => 
        todo.optimistic && todo.title === savedTodo.title 
          ? savedTodo 
          : todo
      )
    )
  }
})
```

### Garbage Collection
```tsx
// Global default
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      cacheTime: 10 * 60 * 1000, // 10 minutes
      staleTime: 5 * 60 * 1000,  // 5 minutes
    },
  },
})

// Per-query override
useQuery(['large-dataset'], fetchLargeData, {
  cacheTime: 60 * 1000, // Clear after 1 minute of inactivity
  staleTime: 0 // Always consider stale
})

// Manual garbage collection
queryClient.removeQueries(['old-data'])
queryClient.clear() // Nuclear option: clear entire cache
```

## Advanced Patterns

### Query Filters
```tsx
// Invalidate with filters
queryClient.invalidateQueries({
  queryKey: ['posts'],
  refetchType: 'active', // Only refetch active queries
  exact: false // Partial match
})

// Reset queries
queryClient.resetQueries(['posts'], { exact: true })

// Remove queries
queryClient.removeQueries(['posts'], { exact: false })

// Get queries programmatically
const activePostQueries = queryClient.getQueriesData({
  queryKey: ['posts'],
  type: 'active'
})
```

### Hydration & SSR
```tsx
// Server-side
const queryClient = new QueryClient()
await queryClient.prefetchQuery(['posts'], fetchPosts)
const dehydratedState = dehydrate(queryClient)

// Send to client
return {
  props: { dehydratedState }
}

// Client-side
function MyApp({ pageProps }) {
  const [queryClient] = useState(() => new QueryClient())
  
  return (
    <QueryClientProvider client={queryClient}>
      <Hydrate state={pageProps.dehydratedState}>
        <Component {...pageProps} />
      </Hydrate>
    </QueryClientProvider>
  )
}
```

### Persister
```tsx
import { persistQueryClient } from '@tanstack/react-query-persist-client'
import { createSyncStoragePersister } from '@tanstack/query-sync-storage-persister'

const persister = createSyncStoragePersister({
  storage: window.localStorage,
})

persistQueryClient({
  queryClient,
  persister,
  maxAge: 1000 * 60 * 60 * 24, // 24 hours
})
```

### Suspense Mode
```tsx
// Enable suspense
useQuery(['user'], fetchUser, {
  suspense: true,
})

// Wrap with Suspense boundary
<Suspense fallback={<Spinner />}>
  <UserProfile />
</Suspense>

// Error handling with suspense
<ErrorBoundary fallback={<Error />}>
  <Suspense fallback={<Spinner />}>
    <UserProfile />
  </Suspense>
</ErrorBoundary>
```

### Window Focus Refetching
```tsx
// Custom focus manager
import { focusManager } from '@tanstack/react-query'

// Pause focus refetching
focusManager.setFocused(false)

// Resume
focusManager.setFocused(true)

// Custom focus detection
focusManager.setEventListener((handleFocus) => {
  if (typeof window !== 'undefined') {
    window.addEventListener('focus', handleFocus)
    return () => window.removeEventListener('focus', handleFocus)
  }
})
```

### Network Status Management
```tsx
// Custom online manager
import { onlineManager } from '@tanstack/react-query'

// Set online status
onlineManager.setOnline(false)

// Custom online detection
onlineManager.setEventListener((setOnline) => {
  // Custom logic
  return window.addEventListener('online', () => setOnline(true))
})
```

## Key Performance Patterns

### 1. Query Invalidation Strategy
```tsx
// Specific invalidation
queryClient.invalidateQueries(['users', userId])

// Broad invalidation
queryClient.invalidateQueries(['users'])

// Selective invalidation
queryClient.invalidateQueries({
  queryKey: ['users'],
  refetchType: 'active', // or 'inactive' or 'all'
  exact: false
})

// Predicate-based
queryClient.invalidateQueries({
  predicate: (query) => 
    query.queryKey[0] === 'posts' && query.state.data?.userId === currentUserId
})
```

### 2. Structural Sharing
- **Enabled by default**: Deep comparison prevents rerenders when data shape identical
- **Disable for large datasets**: `structuralSharing: false` if performance cost > benefit
- **Custom comparison**: Provide your own comparison function
```tsx
useQuery(['data'], fetchData, {
  structuralSharing: (oldData, newData) => {
    // Custom logic
    return isEqual(oldData, newData) ? oldData : newData
  }
})
```

### 3. Query Cancellation Patterns
```tsx
// Automatic cancellation
const queryInfo = useQuery(['search', searchTerm], 
  ({ signal }) => fetch(`/api/search?q=${searchTerm}`, { signal }),
  { enabled: searchTerm.length > 0 }
)

// Manual cancellation
const queryClient = useQueryClient()
queryClient.cancelQueries(['search'])
```

### 4. Dependent Queries
```tsx
// Sequential dependencies
const { data: user } = useQuery(['user'], fetchUser)
const { data: projects } = useQuery(
  ['projects', user?.id],
  () => fetchProjects(user.id),
  { enabled: !!user?.id }
)

// Parallel with dependency check
const queries = useQueries([
  { queryKey: ['user'], queryFn: fetchUser },
  { 
    queryKey: ['posts'], 
    queryFn: fetchPosts,
    enabled: !!userData // Access first query result
  }
])
```

### 5. Data Transformation Layers
```tsx
// Transform at query level
useQuery(['users'], fetchUsers, {
  select: (users) => users.map(normalizeUser)
})

// Transform at mutation level
useMutation(updateUser, {
  onSuccess: (data) => {
    queryClient.setQueryData(['user', data.id], normalizeUser(data))
  }
})

// Global transformation
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      select: (data) => normalizeData(data)
    }
  }
})
```

## Common Anti-Patterns to Avoid

❌ **Over-fetching**: Setting `staleTime: 0` globally  
✅ **Use appropriate staleTime** per query type

❌ **Aggressive invalidation**: Invalidating on every mutation  
✅ **Update cache directly** with `setQueryData` for optimistic updates

❌ **Ignoring query keys**: Not including all dependencies  
✅ **Keys must capture all variables** that affect data

❌ **Synchronous transformations in render**: `data?.map()` in component  
✅ **Use `select` option** for memoized transforms

❌ **Missing error boundaries**: Letting errors bubble unhandled  
✅ **Wrap with ErrorBoundary** or handle `isError` state

❌ **Inline query functions**: Creating new functions every render  
✅ **Define stable query functions** outside component

❌ **Over-polling**: Setting aggressive refetch intervals  
✅ **Use WebSockets/SSE** for real-time data, polling as fallback

❌ **Not using enabled**: Running queries before dependencies ready  
✅ **Gate queries with enabled** option

❌ **Forgetting cleanup**: Not cancelling on unmount  
✅ **React Query handles cleanup** automatically

❌ **Duplicate query keys**: Same key for different data  
✅ **Unique, hierarchical keys** for each data shape

## Debugging & DevTools

### React Query DevTools
```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

<QueryClientProvider client={queryClient}>
  <App />
  <ReactQueryDevtools initialIsOpen={false} />
</QueryClientProvider>
```

### Query State Inspection
```tsx
// Get query state
const queryState = queryClient.getQueryState(['user', id])
console.log(queryState.status) // 'loading' | 'error' | 'success'
console.log(queryState.fetchStatus) // 'fetching' | 'paused' | 'idle'

// Get query data
const userData = queryClient.getQueryData(['user', id])

// Set query data manually
queryClient.setQueryData(['user', id], newUserData)

// Imperatively fetch
const data = await queryClient.fetchQuery(['user', id], fetchUser)
```

### Logging
```tsx
const queryClient = new QueryClient({
  logger: {
    log: (...args) => console.log(...args),
    warn: (...args) => console.warn(...args),
    error: (...args) => console.error(...args),
  },
})

// Custom logging
useQuery(['data'], fetchData, {
  onSuccess: (data) => console.log('Query succeeded', data),
  onError: (error) => console.error('Query failed', error),
  onSettled: (data, error) => console.log('Query settled')
})
```

## Testing Patterns

### Mock Query Client
```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

const createTestQueryClient = () => new QueryClient({
  defaultOptions: {
    queries: { retry: false },
    mutations: { retry: false },
  },
  logger: {
    log: console.log,
    warn: console.warn,
    error: () => {}, // Suppress errors in tests
  },
})

// In test
const queryClient = createTestQueryClient()
render(
  <QueryClientProvider client={queryClient}>
    <Component />
  </QueryClientProvider>
)
```

### Prefill Cache for Tests
```tsx
queryClient.setQueryData(['user'], mockUserData)
```

### Wait for Queries
```tsx
import { waitFor } from '@testing-library/react'

await waitFor(() => expect(screen.getByText('Loaded')).toBeInTheDocument())
```