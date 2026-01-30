# Vite (React/Vue/Svelte)

Deploy Vite-built applications to Buddy Packages.

## Build

```bash
npm run build
```

Output directory: `./dist`

## Deploy

```bash
bdy package publish my-app@1.0.0 ./dist --create
```

## Base Path

If deploying to a subpath, configure `vite.config.js`:

```js
export default {
  base: '/subpath/'
}
```

For root deployment, no changes needed (default base is `/`).

## Verify

```bash
bdy package version get my-app 1.0.0
```

Access your site at the returned URL.
