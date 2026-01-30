# Next.js Static Export

Deploy Next.js static exports to Buddy Packages.

## Configure

Add static export to `next.config.js`:

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export'
}

module.exports = nextConfig
```

## Build

```bash
npm run build
```

Output directory: `./out`

## Deploy

```bash
bdy package publish my-app@1.0.0 ./out --create
```

## Notes

- Static export doesn't support server-side features (API routes, SSR, ISR)
- Image optimization requires external loader or `unoptimized: true`
- Dynamic routes need `generateStaticParams`

## Verify

```bash
bdy package version get my-app 1.0.0
```
