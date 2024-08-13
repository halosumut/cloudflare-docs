---
pcx_content_type: get-started
title: Get started
weight: 1
meta:
  title: Get started | Full-stack (SSR) | Next.js apps
  description: Deploy a full-stack Next.js app to Cloudflare Pages
---

# Next.js

Learn how to deploy full-stack (SSR) Next.js apps to Cloudflare Pages.

## New apps

To create a new Next.js app, pre-configured to run on Cloudflare, run:

```sh
$ npm create cloudflare@latest my-next-app -- --framework=next
```

For more guidance on developing your app, refer to [Bindings](/pages/framework-guides/nextjs/ssr/bindings/) or the [Next.js documentation](https://nextjs.org).

---

## Existing apps


### 1. Install next-on-pages

First, install [@cloudflare/next-on-pages](https://github.com/cloudflare/next-on-pages):

```sh
$ npm install --save-dev @cloudflare/next-on-pages
```

### 2. Add `wrangler.toml` file

Then, add a [`wrangler.toml`](/pages/functions/wrangler-configuration/) file to the root directory of your Next.js app:

```toml
name = "my-app"
compatibility_date = "2024-07-29"
compatibility_flags = ["nodejs_compat"]
pages_build_output_dir = ".vercel/output/static"
```

This is where you configure your Pages project and define what resources it can access via [bindings](/workers/runtime-apis/bindings/).

### 3. Update `next.config.mjs`

Next, update the content in your `next.config.mjs` file.

```diff
---
header: next.config.mjs
---
+ import { setupDevPlatform } from '@cloudflare/next-on-pages/next-dev';

/** @type {import('next').NextConfig} */
const nextConfig = {};

+ if (process.env.NODE_ENV === 'development') {
+   await setupDevPlatform();
+ }

export default nextConfig;
```

These changes allows you to access [bindings](/pages/framework-guides/nextjs/ssr/bindings/) in local development.

### 4. Ensure all server-rendered routes use the Edge Runtime

Next.js has [two "runtimes"](https://nextjs.org/docs/app/building-your-application/rendering/edge-and-nodejs-runtimes) — "Edge" and "Node.js". When you run your Next.js app on Cloudflare, you [can use available Node.js APIs](/workers/runtime-apis/nodejs/) — but you currently can only use Next.js' "Edge" runtime.

This means that for each server-rendered route — ex: an API route or one that uses `getServerSideProps` — you must configure it to use the "Edge" runtime:

```js
export const runtime = "edge";
```

### 5. Update `package.json`

Add the following to the scripts field of your `package.json` file:

```json
---
header: package.json
---
"pages:build": "npx @cloudflare/next-on-pages",
"preview": "npm run pages:build && wrangler pages dev",
"deploy": "npm run pages:build && wrangler pages deploy"
```

- `npm run pages:build`: Runs `next build`, and then transforms its output to be compatible with Cloudflare Pages.
- `npm run preview`: Builds your app, and runs it locally in [workerd](https://github.com/cloudflare/workerd), the open-source Workers Runtime. (`next dev` will only run your app in Node.js)
- `npm run deploy`: Builds your app, and then deploys it to Cloudflare

### 6. Deploy to Cloudflare Pages

Either deploy via the command line:

```sh
$ npm run deploy
```

Or [connect a Github or Gitlab repository](/pages/get-started/git-integration/), and Cloudflare will automatically build and deploy each pull request you merge to your production branch.

### 7. (Optional) Add `eslint-plugin-next-on-pages`

Optionally, you might want to add `eslint-plugin-next-on-pages`, which lints your Next.js app to ensure it is configured correctly to run on Cloudflare Pages.

```sh
$ npm install --save-dev eslint-plugin-next-on-pages
```

Once it is installed, add the following to `.eslintrc.json`:

```diff
---
header: .eslintrc.json
---
{
  "extends": [
    "next/core-web-vitals",
+    "plugin:eslint-plugin-next-on-pages/recommended"
  ],
  "plugins": [
+    "eslint-plugin-next-on-pages"
  ]
}
```

## Related resources

- [Bindings](/pages/framework-guides/nextjs/ssr/bindings/)
- [Troubleshooting](/pages/framework-guides/nextjs/ssr/troubleshooting/)