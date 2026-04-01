# PostHTML-static

This template repository contains scripts and config files for building HTML from a source tree of modular HTML files and serving the resulting HTML files via an Nginx docker container.

**Features**: 
- File System Routing,
- HTML Components via <a href="https://github.com/posthtml/posthtml-include">PostHTML-include</a>,
- ready for serving via docker.

Example source tree:
```
src
├── blog
│   └── index.page.html
├── components
│   ├── footer.comp.html
│   ├── header.comp.html
│   └── nav.comp.html
├── images
│   └── logo.png
└── index.page.html
```

Resulting dist tree (contains non-HTML files and merged HTML modules):
```
dist
├── blog
│   └── index.html
├── images
│   └── logo.png
└── index.html

```

## Include Syntax

```html
<!-- relative to importing file -->
<include src="./button.comp.html"></include>
<include src="button.comp.html"></include>
<!-- absolut to projects ./src dir -->
<include src="/components/header.comp.html"></include>
```

See <a href="https://github.com/posthtml/posthtml-include">PostHTML-include</a> for more details, like parameters.
`./src` contains an example website.

## Usage

Build website:
```bash
# setup
npm install # install build.js dependencies

# build pages once
npm run build
# build pages every few seconds. 
npm run dev
```

Serve changing files from `./dist` via nginx docker container:
```bash
docker compose up -d --build
```
For distribution, comment the `./dist` volume inside the docker compose file and rebuild the container.
This volume allows Nginx to see changes made to files in the `./dist` directory. 
On build time `./dist` is also copied into the container.

## File Conventions

| Pattern | Purpose | Output |
|---------|---------|--------|
| `*.page.html` | Page template | `./dist/` |
| `*.comp.html` | Reusable component | `./build/` |

## How the building process works?

1. Find all HTML files in `src/`
2. Parse each file to extract `<include>` tags
3. Sort files so dependencies build first; abort if circular
4. Process each file:
   - Inline included components
   - Replace `{{ variables }}` with JSON values
   - Cache built components
5. Write output:
   - Pages (`*.page.html`) → `dist/`
   - Components (`*.comp.html`) → `build/` (cache)
   - Other files → `dist/`
   - Static files → copy to `dist/`

