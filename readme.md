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

```javascript

```

- After component mounts and data fetched: `data.pages[0]: {...} & update pageParamZ`
- `hasNExtPage`? -> `fetchNextPage`
- No more pages? -> `pageParam: undefined & hasNextPage: false`
