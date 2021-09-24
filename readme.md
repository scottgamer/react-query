# React Query

This describes the basics on how to use the [React Query Library](https://react-query.tanstack.com/)

## Setting up the library

- Install the library `npm install react-query`
- Create query client:
  - manages queries and cache

```jsx
import { QueryClient, QueryClientProvider } from "react-query";

const queryClient = new QueryClient();

function App() {
  return <QueryClientProvider client={queryClient}>...</QueryClientProvider>;
}

export default App;
```

- Apply QueryProvider:
  - provides cache and client config to children
  - takes query client as the value
- Run `useQuery`
  - hook to query the server

```jsx
import { useQuery } from "react-query";

async function fetchPosts() {
  const response = await fetch(
    "https://jsonplaceholder.typicode.com/posts?_limit=10&_page=0"
  );
  return response.json();
}

const { data } = useQuery("posts", fetchPosts);

return (
  <ul>
    {data?.map((post) => (
      <li key={post.id}>{post.title}</li>
    ))}
  </ul>
);
```

**More on [useQuery](<[useQuery](https://react-query.tanstack.com/reference/useQuery)>)**

## isFetching vs isLoading

- isFetching
  - the async query function hasn't yet resolved
- isLoading
  - no cached data, plus `isFetching`

## React Query Dev Tools

- Shows queries (by key)
  - status of queries
  - last updated timestamp
- Data explorer
- Query explorer
- To add the `react query dev tools`:

```jsx
// App component
import { ReactQueryDevtools } from "react-query/devtools";
return (
  <>
    ...
    <ReactQueryDevtools />
  </>
);
```

**More on [React Query Dev Tools](https://react-query.tanstack.com/devtools)**

## Stale Data

- Data refetch only triggers for stale data
  - component remounts, window refocus
  - `staleTime` translates to "max age"
  - how to tolerate data potentially being out of date
- You can add the `staleTime` in the `options` object of `useQuery`

```javascript
const { data, error, isError, isLoading } = useQuery("posts", fetchPosts, {
  staleTime: 2000,
});
```

- The default `staleTime` is `0` to always have updated data

### staleTime vs cacheTime

- `staleTime` is for re-fetching
- `cache` is for data that might be re-used later
  - query goes into "cold storage" if there's no active `useQuery`
  - cache data expires after `cacheTime` (default: five minutes)
    - how long it's been since the last active `useQuery`
  - after the cache expires, the data is garbage collected
  - cache is backup data to display while fetching

### Why some data doesn't refresh?

- Data for queries with known keys only re-fetched upon trigger
- Example triggers:
  - component remount
  - window refocus
  - running refetch function
  - automated refetch
  - query invalidation after a mutation
- To fix this, it's possible to pass an array for the query, not only a string
- Treat the query as a dependency array `['comments', post.id]`

## Pre-fetching

- adds data to cache
- automatically state (configurable)
- shows while re-fetching
  - as long as cache hasn't expired
- pre-fetching can be used for any anticipated data needs
  - not just pagination

```jsx
import { useEffect, useState } from "react";

import { useQuery, useQueryClient } from "react-query";

export function Posts() {
  const [currentPage, setCurrentPage] = useState(1);

  // the query client is necessary to pre-fetch data
  const queryClient = useQueryClient();

  useEffect(() => {
    // condition to pre-fetch data
    if (currentPage < maxPostPage) {
      const nextPage = currentPage + 1;
      queryClient.prefetchQuery(["posts", nextPage], () =>
        fetchPosts(nextPage)
      );
    }
  }, [currentPage, queryClient]);

  const { data } = useQuery(
    ["posts", currentPage],
    () => fetchPosts(currentPage),
    {
      staleTime: 2000,
      // keep data in cache
      keepPreviousData: true,
    }
  );
}
```

**More on [Pre-fetching](https://react-query.tanstack.com/guides/prefetching)**

## Mutations

- Making a network call that changes data on the server
  - Optimistic updates (assume change will happen)
  - Update react query cache with data returned from the server
  - Trigger re-fetch of relevant data (invalidation)
- `useMutation`
  - similar to `useQuery`
  - returns a `mutate` function
  - doesn't need a query key
  - `isLoading` but no `isFetching`
  - by default, no retries (configurable)

**More on [Mutations](https://react-query.tanstack.com/guides/mutations)**

## Infinite Queries

- `useInfiniteQuery`
- Requires different API format than pagination
- Tracks next query
  - next query is returned as part of the data
- Data object has 2 properties:
  - `pages`
  - `pageParams` tracks the keys of queries that have been retrieved (not commonly used)
- Every query has its own element in the pages array
- Current value of `pageParam` is maintained by react query
- `useInfiniteQuery` options:
  - `getNextPageParam`: (lastPage, allPages)
    - updates `pageParam`
    - might use all of the pages of data (allPages)
    - `lastPage` works as a `next` property
  - `fetchNextPage`
    - function to call when the user needs more data
  - `hasNextPage`
    - based on return value of `getNextPageParam`
    - if `undefined`, no more data
  - `isFetchingNextPage`
    - for displaying loading spinner
    - difference between `isFetching` & `isFetchingNextPage`

### Data Flow

- Component mounts: `data: undefined`
- Component mounts -> Fetch first page: `data: undefined & pageParam: default`
- After component mounts and data fetched: `data.pages[0]: {...} & update pageParamZ`
- `hasNExtPage`? -> `fetchNextPage`
- No more pages? -> `pageParam: undefined & hasNextPage: false`

```jsx
import { useInfiniteQuery } from "react-query";

const initialUrl = "https://dummy/api";

const fetchUrl = async (url) => {
  const response = await fetch(url);
  return response.json();
};

export function InfinitePeople() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isLoading,
    isFetching,
    isError,
    error,
  } = useInfiniteQuery(
    "sw-people",
    ({ pageParam = initialUrl }) => fetchUrl(pageParam),
    { getNextPageParam: (lastPage) => lastPage.next || undefined }
  );

  if (isLoading) {
    return <h3>Loading...</h3>;
  }

  if (isError) {
    return <h3>Error {error.toString()}</h3>;
  }

  return (
    <>
      {isFetching && <h3>Loading...</h3>}
      <InfiniteScroll loadMore={fetchNextPage} hasMore={hasNextPage}>
        {data.pages.map((pageData) =>
          pageData.map((person) => <Person person={person} />)
        )}
      </InfiniteScroll>
    </>
  );
}
```

## Custom Hooks

- Just as in react, custom hooks help abstract logic from a component grouping multiple hooks

```typescript
import { useQuery } from "react-query";

import type { Treatment } from "../../../../../shared/types";
import { axiosInstance } from "../../../axiosInstance";
import { queryKeys } from "../../../react-query/constants";

// for when we need a query function for useQuery
async function getTreatments(): Promise<Treatment[]> {
  const { data } = await axiosInstance.get("/treatments");
  return data;
}

export function useTreatments(): Treatment[] {
  // const fallback = [];
  const { data = [] } = useQuery(queryKeys.treatments, getTreatments);
  return data;
}
```

**Read more on [Custom hooks](https://react-query.tanstack.com/examples/custom-hooks)**

### useIsFetching

- In smaller apps:

  - used **isFetching** from **useQuery** return object
  - **isLoading** is **isFetching** plus no cached data

- In larger apps:

  - loading spinner whenever any query **isFetching**
  - **useIsFetching** helps in this case

- No need for **isFetching** on every custom hook / useQuery call

### Default onError option

- No `useError` analogy for `useFetching`
  - not a boolean: unclear how to implement
- Instead, set default `onError` handler for queryClient

```javascript
{
  queries: { useQuery options },
  mutations: { useMutation options }
}
```

- It's possible to pass a default error handler to the query client constructor
  - this will handle all errors inside the query provider

```typescript
export function queryErrorHandler(error: unknown): void {
  const id = "react-query-error";
  const title =
    error instanceof Error
      ? error.toString().replace(/^Error:\s*/, "")
      : "Error connecting to server";

  toast.closeAll();
  toast({ id, title, status: "error", variant: "subtle", isClosable: true });
}

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      onError: queryErrorHandler,
    },
  },
});
```

- Then use the queryClient without any optional parameters

```typescript
export function useTreatments(): Treatment[] {
  const { data = [] } = useQuery(queryKeys.treatments, getTreatments);
  return data;
}
```

## Why doesn't data refresh?

- using the same key for every query
- nothing to trigger refetch
  - component remount
  - window refocus
  - running refetch function manually
  - automated refetch
- **use keys for every change!**
  - treat keys a dependency array

```typescript
const { data: appointments = fallback } = useQuery(
  [queryKeys.appointments, monthYear.year, monthYear.month],
  () => getAppointments(monthYear.year, monthYear.month)
);
```

## Data pre-fetching

- data can be pre-fetched using the `useQueryClient` hook and the `prefetchQuery` method

```typescript
import { useQuery, useQueryClient } from "react-query";

// prefetch next month when monthYear changes
const queryClient = useQueryClient();
useEffect(() => {
  // assume increment of one month
  const nextMonthYear = getNewMonthYear(monthYear, 1);
  queryClient.prefetchQuery(
    [queryKeys.appointments, nextMonthYear.year, nextMonthYear.month],
    () => getAppointments(nextMonthYear.year, nextMonthYear.month)
  );
}, [queryClient, monthYear]);
```

## The Select Option

- transform or select a part of the data returned by the query function
- it allows to filter out data from `useQuery`
- react query memoizes data to reduce unnecessary computation
- tech details:
  - triple equals `"==="` comparison of `select` function
  - only runs if data changes and the function has changed
- need a stable function (`useCallback` for anonymous function)
- `useCallback` will also improve the performance of the caching method
- **Select is not an option for pre-fetch!**

**More on [Data Transformation](https://tkdodo.eu/blog/react-query-data-transformations)**

```typescript
// imports...

async function getStaff(): Promise<Staff[]> {
  const { data } = await axiosInstance.get("/staff");
  return data;
}

export function useStaff(): UseStaff {
  const [filter, setFilter] = useState("all");
  const selectFn = useCallback(
    (unfilteredStaff) => filterByTreatment(unfilteredStaff, filter),
    [filter]
  );

  const fallback = [];
  // use of the select property to apply filter to cached data
  const { data: staff = fallback } = useQuery(queryKeys.staff, getStaff, {
    select: filter !== "all" ? selectFn : undefined,
  });

  return { staff, filter, setFilter };
}
```

## Refetching

- re-fetch ensures stale data gets updated from server
  - leave page and refocus
- stale queries are re-fetched automatically in the background when:
  - new instance of the query mount
  - every time a react component using react query mounts
  - the window is refocused
  - the network is reconnected
  - configured `refetchInterval` has expired
    - automatic `polling`

### How?

- global or query-specific options:
  - `refetchOnMount`, `refetchOnWindowFocus`
  - `refetchOnReconnect`, `refetchInterval`

### Suppressing re-fetch

- increase stale time
- turn off refetchOnMount / refetchOnWindowFocus / refetchOnReconnect
- only for very rarely changed, not mission-critical data
- suppressing the refetch can be done in each individual query client or in the global query client

```typescript
export function useTreatments(): Treatment[] {
  const { data = [] } = useQuery(queryKeys.treatments, getTreatments, {
    staleTime: 600000, // 10 minutes
    cacheTime: 900000, // 15 minutes (doesn't make sense for staleTime to exceed cacheTime)
    refetchOnMount: false,
    refetchOnWindowFocus: false,
    refetchOnReconnect: false,
  });
  return data;
}
```

```typescript
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      onError: queryErrorHandler,
      staleTime: 10 * 60 * 1000,
      cacheTime: 15 * 60 * 1000,
      refetchOnMount: false,
      refetchOnReconnect: false,
      refetchOnWindowFocus: false,
    },
  },
});
```

## Authentication

### Dependent queries

- dependent queries

## Caching values for logged in users

- without a provider, no persistence across `useUser` calls
- react query acting as a provider for auth
- use `queryClient.setQueryData`
- add to `updateUser` and `clearUser`

```typescript
import { useState } from "react";
import { useQuery, useQueryClient } from "react-query";
import { queryKeys } from "../../../react-query/constants";
import {
  clearStoredUser,
  getStoredUser,
  setStoredUser,
} from "../../../user-storage";

async function getUser(user: User | null): Promise<User | null> {
  if (!user) return null;
  const { data } = await axiosInstance.get(`/user/${user.id}`, {
    headers: getJWTHeader(user),
  });
  return data.user;
}

export function useUser(): UseUser {
  const [user, setUser] = useState<User | null>(getStoredUser());
  const queryClient = useQueryClient();

  // call useQuery to update user data from server
  useQuery(queryKeys.user, () => getUser(user), {
    enabled: !!user,
    onSuccess: (data) => setUser(data),
  });

  // meant to be called from useAuth
  function updateUser(newUser: User): void {
    // set user in state
    setUser(newUser);
    // update user in localstorage
    setStoredUser(newUser);
    // pre-populate user profile in React Query client
    queryClient.setQueryData(queryKeys.user, newUser);
  }

  // meant to be called from useAuth
  function clearUser() {
    // update state
    setUser(null);
    // remove from localstorage
    clearStoredUser();
    // reset user to null in query client
    queryClient.setQueryData(queryKeys.user, null);
  }

  return { user, updateUser, clearUser };
}
```

- in this case, after the user data has been fetched, it is set to the query client using the `setQueryData` method

- `useQuery` caches user data and refreshes from server
  - refreshing from server will be important for mutations
- `useUser` manages user data in query cache and `localStorage`
  - set query cache using `setQueryData` on sign in / sign out
- user `useQuery` dependent on `user` state being truthy
  - `user` state initially set by `updateUser` called by auth `signin`
  - can't query server if we don;t have a user ID!
  - can't remove query because query writes to user state
