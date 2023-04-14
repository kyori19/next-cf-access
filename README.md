# next-cf-access

Protect your Next.js + Vercel project with Cloudflare Access including preview deployments.

## Getting Started

### Prerequisites

* [Next.js](https://nextjs.org/) project you want to protect
  * with AppDir support
* [Vercel](https://vercel.com/) account with a project
  * Custom domain configured
* [Cloudflare](https://www.cloudflare.com/) account
* Your own domain name on Cloudflare

### Install the package

Install [next-cf-access](https://www.npmjs.com/package/next-cf-access) package inside your Next.js project.

```sh
yarn add next-cf-access
```

### Configure the API route

next-cf-access needs 2 API routes under `/.ncfa/`.

Create `app/.ncfa/authorize/route.ts` (or `route.js` if you prefer JavaScript) with the following content:

```ts
export * from 'next-cf-access/app/ncfa/authorize';
```

Create `app/.ncfa/redirect/route.ts` (or `route.js` if you prefer JavaScript) with the following content:

```ts
export * from 'next-cf-access/app/ncfa/redirect';
```

(Optional) If you want to try protecting preview deployments ASAP, deploy the project to **Production** environment on Vercel once here.

### Protect your contents

Change `app/layout.tsx` (or `layout.jsx` if you prefer JavaScript) to the following:

```tsx
import { useAccess } from 'next-cf-access';
import { Redirect } from 'next-cf-access/components';

// Must add `async` on RootLayout
export default async function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  // Add this line
  const result = await useAccess();

  return (
    <html lang="en">
      {/** Change the contents of <body> to this trinary operator */}
      {/** You must not return contents when `result && result.err` is true */}
      <body>{result && result.err ? <Redirect /> : children}</body>
    </html>
  )
}
```

### Configure Cloudflare Access

Login to [Cloudflare Zero Trust dashboard](https://dash.teams.cloudflare.com/).

Create a new application with the custom domain you configured on Vercel.
You can configure session duration and other settings as you like.
Save the application and get the **Application Audience (AUD) tag** from Application Configuration.

**Additionally**, you need to create one more application for well-known endpoints.
For this application, you must use the same domain as the previous application, and set **path** to `.well-known`.
Furthermore, you must add the policy with **Action: Bypass** and **Everyone** as **Include** rules.

### Issue Vercel Access Token

Login to Vercel and open [Access Tokens](https://vercel.com/account/tokens) page.
Create and save a new token.
Scope must be **Full Account** if you are deploying to your personal account.

### Configure Environment Variables

You need to set the following environment variables on Vercel.
All variables must be exposed to the all environments including **Production**, **Preview**, and **Development**.

* `NCFA_AUD` - Application Audience (AUD) tag from Cloudflare Access Application Configuration
* `NEXT_PUBLIC_NCFA_DOMAIN` - Custom domain name of your Next.js project on Vercel
* `VERCEL_TOKEN` - Vercel Access Token

### Deploy to Vercel

Deploy your project to Vercel.
Your project will now be protected by Cloudflare Access including Preview Deployments.
