# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

YdHelper is a Vue 3 application built with Vite. This is a minimal starter template using Vue 3 Composition API with `<script setup>` syntax.

## Development Commands

```sh
npm install          # Install dependencies
npm run dev          # Start development server with hot-reload
npm run build        # Compile and minify for production
npm run preview      # Preview production build locally
```

## Code Architecture

### Project Structure

- `src/App.vue` - Root application component using Composition API with `<script setup>`
- `src/main.js` - Application entry point that creates and mounts the Vue app
- `index.html` - HTML entry point that mounts the Vue app to `#app`
- `vite.config.js` - Vite configuration including path aliases and plugins

### Key Configuration Details

**Path Aliases**: The `@` symbol is aliased to `./src` directory in both `vite.config.js` and `jsconfig.json`. Use `@/` prefix for imports from the src directory.

**Vue DevTools**: The project includes `vite-plugin-vue-devtools` for Vue 3 debugging in Chromium-based browsers and Firefox.

**Node Version**: Requires Node.js `^20.19.0 || >=22.12.0` (specified in package.json engines field).

### Vue Setup

- Uses Vue 3.5+ with Composition API
- Components use `<script setup>` syntax by default
- Single File Components (`.vue`) follow the standard `<script setup>`, `<template>`, `<style scoped>` structure
