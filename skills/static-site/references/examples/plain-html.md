# Plain HTML/CSS/JS

Deploy static HTML files to Buddy Packages.

## No Build Step

Static HTML sites don't require a build step.

## Deploy

From project root containing `index.html`:

```bash
bdy package publish my-site@1.0.0 . --create
```

From a subdirectory (e.g., `./public`):

```bash
bdy package publish my-site@1.0.0 ./public --create
```

## Directory Structure

Typical structure:
```
my-site/
├── index.html
├── css/
│   └── style.css
├── js/
│   └── main.js
└── images/
    └── logo.png
```

## Verify

```bash
bdy package version get my-site 1.0.0
```

Access your site at the returned URL.
