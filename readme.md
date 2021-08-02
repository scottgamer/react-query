# React Query

This describes the basics on how to use the [React Query Library](https://react-query.tanstack.com/)

## Setting up the library

- Install the library `npm install react-query`
- Create query client:
  - manages queries and cache
- Apply QueryProvider:
  - provides cache and client config to children
  - takes query client as the value
- Run `useQuery`
  - hook to query the server
