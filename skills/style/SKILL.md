---
name: style
description: Install @shaunburdick's personal style configuration. Sets up the shared .editorconfig for any project, and installs eslint-config-shaunburdick for JavaScript/TypeScript projects. Load this skill when setting up a new project, when asked to "add linting", "set up code style", "configure eslint", "add prettier", "set up formatting", "configure editorconfig", or "standardize code style" — even if the user doesn't mention a specific tool by name.
license: MIT
metadata:
  author: shaunburdick
  version: "1.0.0"
---

# Style Setup

Install @shaunburdick's personal style configuration from [github.com/shaunburdick/style](https://github.com/shaunburdick/style).

## Steps

1. Check whether a `.editorconfig` file already exists in the project root. If it does, show the user the existing content and confirm before overwriting.

2. Fetch and follow the install instructions in the root `README.md` at `https://raw.githubusercontent.com/shaunburdick/style/main/README.md`

3. If the project contains JavaScript or TypeScript files (look for `.js`, `.mjs`, `.cjs`, `.ts`, `.tsx`, `.jsx` files, or a `package.json`), also fetch and follow the install instructions in `https://raw.githubusercontent.com/shaunburdick/style/main/eslint/README.md`
