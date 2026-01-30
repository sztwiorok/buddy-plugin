# Plain HTML/CSS/JS

Deploy static HTML files to Buddy Packages.

## No Build Step

Static HTML sites don't require a build step.

## Deploy

```bash
# Create package (first time only)
bdy package create -i my-site

# Publish from project root containing index.html
bdy package publish my-site@1.0.0 .

# Or from a subdirectory (e.g., ./public)
bdy package publish my-site@1.0.0 ./public
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
