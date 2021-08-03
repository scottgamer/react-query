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
