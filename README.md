<h1 align="center">
  🦅 Eagle Multi-Stack Scaffolder
</h1>

<p align="center">
  <strong>Research-driven project scaffolding for Claude Code</strong>
</p>

<p align="center">
  <a href="#-supported-stacks">Stacks</a> •
  <a href="#-installation">Installation</a> •
  <a href="#-usage">Usage</a> •
  <a href="#-how-it-works">How It Works</a> •
  <a href="#-contributing">Contributing</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-Plugin-blueviolet?style=for-the-badge" alt="Claude Code Plugin">
  <img src="https://img.shields.io/badge/Stacks-13+-green?style=for-the-badge" alt="13+ Stacks">
  <img src="https://img.shields.io/badge/Skills-2-orange?style=for-the-badge" alt="2 Skills">
  <img src="https://img.shields.io/badge/License-MIT-blue?style=for-the-badge" alt="MIT License">
</p>

---

## 🤔 The Problem

Starting a new project is painful:
- Which libraries are still maintained?
- What's the recommended project structure in 2026?
- What package manager should I use?
- How do I set up TypeScript/linting/testing properly?

**You end up spending hours researching instead of building.**

## 💡 The Solution

Eagle Multi-Stack Scaffolder teaches Claude to **research before coding**. It:

1. 🔍 **Searches the web** for current best practices
2. 📦 **Recommends packages** based on actual research (not outdated training data)
3. 🏗️ **Generates proper structures** following community standards
4. 📝 **Documents everything** so you know WHY each choice was made

---

## 📱 Supported Stacks

### Mobile Development
| Stack | Use Case |
|-------|----------|
| **SwiftUI** | Native iOS/macOS apps |
| **Jetpack Compose** | Native Android apps (modern) |
| **Kotlin XML Views** | Native Android apps (traditional) |
| **Flutter** | Cross-platform (Dart) |
| **Expo React Native** | Cross-platform (JavaScript/TypeScript) |

All mobile stacks include **design system patterns** with tokens, components, Material 3/Cupertino styling, and dark mode support.

### Backend Development
| Stack | Use Case |
|-------|----------|
| **Node.js + Express** | JavaScript/TypeScript APIs |
| **FastAPI** | High-performance async Python APIs |
| **Flask** | Lightweight Python APIs & microservices |
| **Django** | Python APIs with batteries included |
| **Rust (Axum)** | High-performance, memory-safe APIs |

All backend stacks include:
- **Database setup**: PostgreSQL (primary), MongoDB, SQLite
- **ORM options**: Prisma, Drizzle, SQLAlchemy, SQLModel, SQLx
- **Object storage**: MinIO (S3-compatible)
- **Deployment**: Railway with health checks

### Web Frontend
| Stack | Use Case |
|-------|----------|
| **Next.js** | React SSR/SSG dashboards |
| **Nuxt.js** | Vue SSR/SSG dashboards |

### Architecture
| Stack | Use Case |
|-------|----------|
| **Monorepo (Turborepo)** | Multi-app projects with shared packages |

Monorepo support includes Turborepo + pnpm workspaces, shared packages, remote caching, and per-app Railway deployment.

### Progressive Web Apps (NEW!)
| Stack | Use Case |
|-------|----------|
| **Quasar (Vue)** | Full-featured PWAs with best offline support |
| **Ionic Vue** | PWAs with Capacitor for potential native bridge |
| **Ionic React** | React-based PWAs with native-like UI |
| **Next.js + Konsta UI** | SSR/SEO-focused PWAs |

The **pwa-mobile-first** skill includes:
- **Workbox** service worker configuration
- **Dexie.js** offline data sync with IndexedDB
- **iOS safe-area** handling for notch/home indicator
- **Add-to-Home-Screen** prompts (with iOS fallback)
- **Optimistic UI**, pull-to-refresh, skeleton loaders
- **PWA validation checklist** with automated script

---

## 🚀 Installation

### Step 1: Add the marketplace

```bash
/plugin marketplace add eagleisbatman/eagle-multi-stack-scaffolder
```

### Step 2: Install the plugin

```bash
/plugin install eagle-multi-stack-scaffolder
```

That's it! Claude will now automatically use this skill when relevant.

### Updating the Plugin

To get the latest version with new skills and improvements:

```bash
# Navigate to your Claude Code plugins directory
cd ~/.claude/plugins/eagle-multi-stack-scaffolder

# Pull the latest changes
git pull origin main
```

Or reinstall completely:
```bash
/plugin uninstall eagle-multi-stack-scaffolder
/plugin install eagle-multi-stack-scaffolder
```

---

## 💡 Usage

Just describe what you want to build. Claude will figure out which stacks you need.

### Examples

#### Mobile Development

**iOS app with SwiftUI:**
```
Create a native iOS app for task management with SwiftUI and a proper design system
```

**Android app with Jetpack Compose:**
```
Build a native Android app using Jetpack Compose with Material 3 theming
```

**Android app with XML Views:**
```
Set up a traditional Android app using Kotlin and XML views with Material 3
```

**Flutter cross-platform:**
```
Create a Flutter app for iOS and Android with Riverpod state management and Material 3
```

**React Native with Expo:**
```
Build a cross-platform mobile app using Expo with NativeWind for styling
```

#### Backend Development

**Node.js + Express:**
```
Set up a Node.js Express API with Prisma, PostgreSQL, and JWT authentication
```

**FastAPI (Python):**
```
Create a FastAPI backend with SQLModel, PostgreSQL, Alembic migrations, and MinIO for file uploads
```

**Flask (Python):**
```
Build a Flask REST API with SQLAlchemy, PostgreSQL, and Marshmallow validation
```

**Django (Python):**
```
Set up a Django REST API with PostgreSQL, custom user model, and drf-spectacular docs
```

**Rust with Axum:**
```
Create a high-performance Rust API using Axum with SQLx and PostgreSQL
```

#### Web Frontend

**Next.js dashboard:**
```
Build a Next.js admin dashboard with authentication and server components
```

**Nuxt.js application:**
```
Create a Nuxt.js web app with Vue 3, Pinia state management, and SSR
```

#### Progressive Web Apps

**Offline-first PWA:**
```
Build a PWA for field workers that works offline, syncs data when back online, and feels native on iOS/Android
```

**PWA with Vue:**
```
Create a Quasar PWA with offline data storage, background sync, and add-to-homescreen prompt
```

**SEO-focused PWA:**
```
Build a Next.js PWA with Konsta UI that has good SEO and works offline
```

#### Full-Stack & Multi-Stack

**Full-stack with database:**
```
I need a mobile app with a FastAPI backend, PostgreSQL database, and file uploads to MinIO
```

**Multi-platform project:**
```
Build me:
- React Native app (Expo) with NativeWind
- Express API with Drizzle ORM
- Next.js admin panel
- Shared TypeScript types
```

**Native apps with shared backend:**
```
Create:
- SwiftUI iOS app
- Jetpack Compose Android app
- FastAPI backend with PostgreSQL
```

#### Architecture & DevOps

**Monorepo setup:**
```
Create a Turborepo monorepo with a Next.js web app, Express API, shared UI components, and Railway deployment
```

**Full monorepo with mobile:**
```
Set up a pnpm monorepo with:
- Expo React Native app
- Next.js web app
- Node.js API
- Shared packages for types, utils, and UI
```

---

## ⚙️ How It Works

### 1. Stack Detection
Claude identifies which stacks you need based on your request.

### 2. Research Phase
For **each** stack, Claude searches the web for:
- `"[stack] best practices 2025 2026"`
- `"[stack] recommended libraries 2025"`
- `"[stack] project structure"`

### 3. Documentation Generation
Creates separate docs per stack:

```
your-project/
├── expo/
│   ├── RESEARCH.md       # What Claude found online
│   ├── DEPENDENCIES.md   # Packages with justifications
│   ├── STRUCTURE.md      # Folder organization
│   └── SETUP.md          # Step-by-step commands
├── backend/
│   ├── RESEARCH.md
│   ├── DEPENDENCIES.md
│   └── ...
└── web/
    └── ...
```

### 4. Code Scaffolding
Generates actual project files following the researched best practices.

---

## 📦 Package Manager Philosophy

Like how **Bun** elegantly solves JavaScript dependencies, this plugin recommends the best package manager per ecosystem:

| Ecosystem | Recommended | Why |
|-----------|-------------|-----|
| JavaScript/TypeScript | **Bun** | 10-25x faster, built-in TS |
| Python | **uv** | 10-100x faster than pip |
| Rust | **cargo** | Standard, excellent |
| Swift | **SPM** | Apple native |
| Android | **Version Catalogs** | Modern Gradle standard |

---

## 📁 Reference Files

The plugin includes detailed guides for each stack:

### Multi-Stack Scaffolder
```
skills/eagle-multi-stack-scaffolder/references/
├── swiftui.md           # iOS/macOS with design system
├── jetpack-compose.md   # Android Compose with Material 3
├── kotlin-xml-views.md  # Android XML with Material 3
├── flutter.md           # Flutter with design tokens
├── expo-react-native.md # React Native with NativeWind/Paper
├── node-express.md      # Node.js APIs with Prisma/Drizzle
├── fastapi.md           # FastAPI with SQLModel/SQLAlchemy
├── flask.md             # Flask with Flask-SQLAlchemy
├── django.md            # Django with Django ORM
├── rust-backend.md      # Rust APIs with SQLx
├── nextjs.md            # React web
├── nuxtjs.md            # Vue web
└── monorepo.md          # Turborepo + pnpm workspaces
```

### PWA Mobile-First
```
skills/pwa-mobile-first/
├── SKILL.md                    # Main skill definition
├── references/
│   ├── quasar.md               # Quasar Framework (recommended)
│   ├── ionic-vue.md            # Ionic + Vue
│   ├── ionic-react.md          # Ionic + React
│   └── nextjs-konsta.md        # Next.js + Konsta UI
└── templates/
    ├── manifest.md             # Web app manifest config
    ├── service-worker.md       # Workbox caching strategies
    ├── ios-safe-areas.md       # iOS notch/safe area CSS
    ├── touch-styles.md         # Touch-optimized styles
    ├── offline-sync.md         # Dexie.js + background sync
    ├── a2hs-prompt.md          # Add-to-home-screen component
    ├── workflow-patterns.md    # Optimistic UI, pull-to-refresh
    └── pwa-checklist.md        # Validation checklist + script
```

Each reference includes:
- Research queries to run
- Recommended project structure
- Database & ORM setup (where applicable)
- Essential libraries
- Code patterns
- Deployment configuration
- Setup commands

---

## 🤝 Contributing

Contributions are welcome! Here's how:

1. **Fork** this repository
2. **Improve** a stack reference or add a new one
3. **Test** with Claude Code
4. **Submit** a PR

### Ideas for contributions:
- Add Go backend reference
- Add SvelteKit reference
- Improve existing stack patterns
- Add more community package recommendations

---

## 📄 License

MIT License - use it however you want.

---

<p align="center">
  <strong>Built with 🦅 by <a href="https://github.com/eagleisbatman">@eagleisbatman</a></strong>
</p>

<p align="center">
  <sub>If this helps you, consider giving it a ⭐</sub>
</p>
