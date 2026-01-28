# Expo React Native Reference

## Research Queries
- "Expo SDK 52 new features 2025 2026"
- "React Native best practices 2025"
- "Expo Router file-based routing"
- "NativeWind vs Tamagui vs React Native Paper 2025"
- "React Native community packages popular 2025"

## Package Manager
**Bun** - 10-25x faster installs, deterministic lockfile.

```bash
bunx create-expo-app@latest {app-name} --template tabs
cd {app-name}
bun install
```

## Project Structure (Expo Router)

```
{app-name}/
├── app/                      # Expo Router pages
│   ├── _layout.tsx           # Root layout
│   ├── index.tsx             # Home (/)
│   ├── (tabs)/               # Tab group
│   │   ├── _layout.tsx
│   │   ├── index.tsx
│   │   └── profile.tsx
│   ├── (auth)/               # Auth group
│   │   ├── login.tsx
│   │   └── register.tsx
│   └── [id].tsx              # Dynamic route
├── src/
│   ├── components/
│   │   ├── ui/               # Base components
│   │   └── features/         # Feature components
│   ├── features/
│   │   └── {feature}/
│   │       ├── components/
│   │       ├── hooks/
│   │       └── services/
│   ├── lib/
│   │   ├── api/
│   │   ├── storage/
│   │   └── i18n/
│   ├── hooks/
│   ├── stores/               # Zustand stores
│   └── types/
├── assets/
├── app.json
├── eas.json
├── tailwind.config.js        # NativeWind
└── package.json
```

## UI Framework: NativeWind (Recommended)

```bash
bun add nativewind tailwindcss
bunx tailwindcss init
```

```javascript
// tailwind.config.js
module.exports = {
  content: ["./app/**/*.{js,tsx}", "./src/**/*.{js,tsx}"],
  presets: [require("nativewind/preset")],
}
```

```tsx
// Usage
<View className="bg-white rounded-xl p-4 shadow-md">
  <Text className="text-lg font-semibold">Hello</Text>
</View>
```

## Essential Libraries

| Category | Package | Install |
|----------|---------|---------|
| State | zustand | `bun add zustand` |
| Server State | @tanstack/react-query | `bun add @tanstack/react-query` |
| Forms | react-hook-form + zod | `bun add react-hook-form zod @hookform/resolvers` |
| API | axios | `bun add axios` |
| Images | expo-image | `bun add expo-image` |
| Auth | expo-secure-store | `bun add expo-secure-store` |
| Animations | react-native-reanimated | `bun add react-native-reanimated` |
| Voice | expo-av, expo-speech | `bun add expo-av expo-speech` |

## Community Components to Research

Search before each project:
- "React Native bottom sheet library 2025" → @gorhom/bottom-sheet
- "React Native charts 2025" → victory-native
- "React Native toast 2025" → burnt, react-native-toast-message
- "React Native date picker 2025" → react-native-date-picker

## Code Patterns

### Zustand Store
```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface AuthState {
  user: User | null;
  setUser: (user: User) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      setUser: (user) => set({ user }),
      logout: () => set({ user: null }),
    }),
    { name: 'auth', storage: createJSONStorage(() => AsyncStorage) }
  )
);
```

### React Query Hook
```typescript
export function useFarmers() {
  return useQuery({
    queryKey: ['farmers'],
    queryFn: farmerService.getAll,
    staleTime: 5 * 60 * 1000,
  });
}
```

### Screen Pattern
```tsx
export default function FarmersScreen() {
  const { data, isLoading, refetch } = useFarmers();
  
  return (
    <>
      <Stack.Screen options={{ title: 'Farmers' }} />
      <View className="flex-1 bg-gray-50">
        {isLoading ? <Loading /> : <FarmerList data={data} />}
      </View>
    </>
  );
}
```

## Setup Commands

```bash
# Create
bunx create-expo-app@latest my-app --template tabs
cd my-app

# UI
bun add nativewind tailwindcss
bunx tailwindcss init

# State
bun add zustand @tanstack/react-query axios

# Forms
bun add react-hook-form zod @hookform/resolvers

# EAS Build
bunx eas-cli@latest build:configure

# Run
bun start
```

## EAS Build Config

```json
// eas.json
{
  "build": {
    "development": { "developmentClient": true, "distribution": "internal" },
    "preview": { "distribution": "internal" },
    "production": { "autoIncrement": true }
  }
}
```
