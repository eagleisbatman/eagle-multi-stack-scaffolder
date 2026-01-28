---
name: eagle-multi-stack-scaffolder
description: >
  Research-driven project scaffolding for multiple technology stacks including mobile 
  (SwiftUI, Jetpack Compose, Expo React Native), backend (Node/Express, Django, Rust), 
  and web (Next.js, Nuxt.js). Use this skill when creating new mobile apps for iOS, 
  Android, or cross-platform. Use when setting up backend APIs with Node, Python, or Rust.
  Use when building web dashboards with Next.js or Nuxt.js. Use when scaffolding 
  multi-stack projects. Triggers on phrases like create an app, scaffold a project, 
  set up a new project, or when user needs help with project structure and dependencies.
---

# Eagle Multi-Stack Scaffolder

Research-driven, best-practice scaffolding for multiple technology stacks.

## Core Philosophy

1. **Research-First**: Always search the web for current best practices before scaffolding
2. **Bun-Style Dependencies**: Fast installs, lock files, documented package choices
3. **Separate Artifacts**: Each stack gets its own documentation set

## Stack Selection

| User Request | Stack | Reference |
|--------------|-------|-----------|
| iOS/Apple/Swift | SwiftUI | `references/swiftui.md` |
| Android/Kotlin | Jetpack Compose | `references/jetpack-compose.md` |
| Cross-platform mobile | Expo React Native | `references/expo-react-native.md` |
| Node/Express API | Node.js | `references/node-express.md` |
| Python/Django API | Django | `references/django.md` |
| Rust backend | Rust/Axum | `references/rust-backend.md` |
| React dashboard | Next.js | `references/nextjs.md` |
| Vue dashboard | Nuxt.js | `references/nuxtjs.md` |

## Workflow

### 1. Identify Stacks
Parse user request → determine required stacks (may be multiple).

### 2. Read References
For each stack: `view references/{stack}.md`

### 3. Web Research
For EACH stack, search:
- "[stack] best practices 2025 2026"
- "[stack] recommended libraries"
- "[stack] project structure"

### 4. Generate Artifacts
Per stack, create:
- `RESEARCH.md` - Findings
- `DEPENDENCIES.md` - Packages + justifications
- `STRUCTURE.md` - Folder organization
- `SETUP.md` - Init commands

### 5. Scaffold Code
Generate project structure per reference file patterns.

## Package Managers

| Stack | Recommended |
|-------|-------------|
| JS/TS (Expo, Node, Next, Nuxt) | Bun |
| Python (Django) | uv |
| Rust | cargo |
| Swift | SPM |
| Android | Gradle Version Catalogs |

## Multi-Stack Structure

```
project/
├── mobile/
│   └── expo/
├── backend/
│   └── node/
├── web/
│   └── dashboard/
└── docs/
```

## Rules

1. ALWAYS read reference file before scaffolding
2. ALWAYS search web for current practices
3. NEVER recommend deprecated packages
4. ALWAYS document dependency choices
5. ALWAYS provide working setup commands
