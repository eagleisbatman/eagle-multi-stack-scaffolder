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
  <img src="https://img.shields.io/badge/Stacks-8-green?style=for-the-badge" alt="8 Stacks">
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
| **Jetpack Compose** | Native Android apps |
| **Expo React Native** | Cross-platform mobile apps |

### Backend Development
| Stack | Use Case |
|-------|----------|
| **Node.js + Express** | JavaScript/TypeScript APIs |
| **Django** | Python APIs with batteries included |
| **Rust (Axum)** | High-performance, memory-safe APIs |

### Web Frontend
| Stack | Use Case |
|-------|----------|
| **Next.js** | React SSR/SSG dashboards |
| **Nuxt.js** | Vue SSR/SSG dashboards |

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

---

## 💡 Usage

Just describe what you want to build. Claude will figure out which stacks you need.

### Examples

**Cross-platform mobile app:**
```
Create a fitness tracking app for iOS and Android
```

**Full-stack project:**
```
I need a mobile app with a Node backend and admin dashboard
```

**Specific stack:**
```
Set up a new Django REST API with PostgreSQL
```

**Multi-stack:**
```
Build me:
- React Native app (Expo)
- Express API backend  
- Next.js admin panel
```

---

## ⚙️ How It Works

### 1. Stack Detection
Claude identifies which stacks you need based on your request.

### 2. Research Phase
For **each** stack, Claude searches the web for:
- `"[stack] best practices 2025"`
- `"[stack] recommended libraries"`
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

```
skills/eagle-multi-stack-scaffolder/references/
├── swiftui.md           # iOS/macOS patterns
├── jetpack-compose.md   # Android patterns
├── expo-react-native.md # Cross-platform mobile
├── node-express.md      # Node.js APIs
├── django.md            # Python APIs
├── rust-backend.md      # Rust APIs
├── nextjs.md            # React web
└── nuxtjs.md            # Vue web
```

Each reference includes:
- Research queries to run
- Recommended project structure
- Essential libraries
- Code patterns
- Setup commands

---

## 🤝 Contributing

Contributions are welcome! Here's how:

1. **Fork** this repository
2. **Improve** a stack reference or add a new one
3. **Test** with Claude Code
4. **Submit** a PR

### Ideas for contributions:
- Add Flutter support
- Add Go backend reference
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
