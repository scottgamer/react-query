# React Query

This describes the basics on how to use the [React Query Library](https://react-query.tanstack.com/)

## Setting up the library

- Install the library `npm install react-query`
- Create query client:
  - manages queries and cache

```javascript
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

```javascript
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
