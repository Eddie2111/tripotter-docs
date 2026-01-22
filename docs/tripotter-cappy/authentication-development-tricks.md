---
sidebar_position: 5
---

# Authentication Development Tricks

Tricks you can use to make your authentication flow development easier.

## Mocking the authentication flow

You can mock the authentication flow by using the `mock` parameter in the URL.

For example:

```
http://localhost:3000/?mock=true
```

This will mock the authentication flow and allow you to test the authentication flow without having to go through the actual authentication flow.

## Deploy your site

Test your production build locally:

```bash
npm run serve
```

The `build` folder is now served at [http://localhost:3000/](http://localhost:3000/).

You can now deploy the `build` folder **almost anywhere** easily, **for free** or very small cost (read the **[Deployment Guide](https://docusaurus.io/docs/deployment)**).
