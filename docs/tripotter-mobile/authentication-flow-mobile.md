---
sidebar_position: 2
---

# Authentication Flow

The authentication flow for tripotter mobile is handled by JWT.

## Flow Diagram

![Authentication Flow](/img/docusaurus-social-card.jpg)

## JWT Flow

1. User logs in with their credentials.
2. The server validates the credentials.
3. If the credentials are valid, the server generates a JWT and sends it to the client.
4. The client stores the JWT in the local storage.
5. The client sends the JWT in the Authorization header of every request to the server.
6. The server validates the JWT and allows the request if it is valid.

Add **Markdown or React** files to `src/pages` to create a **standalone page**:

- `src/pages/index.js` → `localhost:3000/`
- `src/pages/foo.md` → `localhost:3000/foo`
- `src/pages/foo/bar.js` → `localhost:3000/foo/bar`

## Create your first React Page

Create a file at `src/pages/my-react-page.js`:

```jsx title="src/pages/my-react-page.js"
import React from 'react';
import Layout from '@theme/Layout';

export default function MyReactPage() {
  return (
    <Layout>
      <h1>My React page</h1>
      <p>This is a React page</p>
    </Layout>
  );
}
```

A new page is now available at [http://localhost:3000/my-react-page](http://localhost:3000/my-react-page).

## Create your first Markdown Page

Create a file at `src/pages/my-markdown-page.md`:

```mdx title="src/pages/my-markdown-page.md"
# My Markdown page

This is a Markdown page
```

A new page is now available at [http://localhost:3000/my-markdown-page](http://localhost:3000/my-markdown-page).
