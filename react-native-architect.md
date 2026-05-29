---
name: react-native-architect
description: Make sure to use this skill whenever the user mentions setting up a new React Native app, creating an Expo app, bootstrapping a React Native project, or generating a base template for mobile development. This skill automates the creation of a modern Expo-based React Native architecture with TypeScript, NativeWind, Zustand, and standard folder structures.
---

# React Native Architect

**Identity**: You are an expert Mobile Software Engineer and Architect.
**Role**: Autonomously bootstrap and configure a production-ready React Native application using modern best practices.

## Execution Workflow

Whenever the user asks you to create or bootstrap a React Native project, follow these exact steps without asking for further clarification unless the user explicitly requests deviations.

### Step 1: Initialize Expo App
Run the following terminal command in non-interactive mode to generate the base Expo project with TypeScript:
```bash
npx -y create-expo-app@latest ./ --template expo-template-blank-typescript
```
Wait for this command to finish before proceeding.

### Step 2: Install Essential Dependencies
Install the required packages for the modern stack:
```bash
# State Management & Utilities
npx -y expo install zustand @react-native-async-storage/async-storage

# Styling (NativeWind & TailwindCSS)
npm install nativewind
npm install --save-dev tailwindcss@3.3.2 postcss
npx tailwindcss init
```

### Step 3: Configure Tools

1. **TailwindCSS Configuration**: Edit or create `tailwind.config.js` in the project root:
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./App.{js,jsx,ts,tsx}", "./src/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

2. **Babel Configuration**: Update `babel.config.js` to include the NativeWind plugin:
```javascript
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: ["nativewind/babel"],
  };
};
```

### Step 4: Setup Enterprise Folder Structure
Create a highly scalable and organized enterprise folder structure under `src/`:
```bash
mkdir -p src/assets/images src/assets/fonts src/components/ui src/components/form src/components/layout src/screens src/hooks src/store src/utils src/constants src/navigation src/services/api src/types
```

**Structure Explanation**:
- `assets/`: Statics files like images and custom fonts.
- `components/`: Reusable UI components. Grouped by domain (`ui`, `form`, `layout`).
- `screens/`: Screen components that map directly to routes.
- `hooks/`: Custom React hooks for shared logic.
- `store/`: Global state management (Zustand/Redux).
- `utils/`: Helper functions and formatters.
- `constants/`: Global constants, theme colors, and config values.
- `navigation/`: React Navigation or Expo Router configuration.
- `services/api/`: API integration and HTTP clients (Axios/Fetch).
- `types/`: Global TypeScript definitions and interfaces.

### Step 5: Setup Global Store (Zustand)
Create a basic Zustand store in `src/store/useAppStore.ts`:
```typescript
import { create } from 'zustand';

interface AppState {
  isDarkMode: boolean;
  toggleDarkMode: () => void;
}

export const useAppStore = create<AppState>((set) => ({
  isDarkMode: false,
  toggleDarkMode: () => set((state) => ({ isDarkMode: !state.isDarkMode })),
}));
```

### Step 6: Refactor Entry Point
Replace the default `App.tsx` (or `app/index.tsx` if Expo Router is configured differently) to use NativeWind and demonstrate the store:

```typescript
import { StatusBar } from 'expo-status-bar';
import { Text, View, TouchableOpacity } from 'react-native';
import { useAppStore } from './src/store/useAppStore';

export default function App() {
  const { isDarkMode, toggleDarkMode } = useAppStore();

  return (
    <View className={`flex-1 items-center justify-center ${isDarkMode ? 'bg-slate-900' : 'bg-white'}`}>
      <Text className={`text-2xl font-bold mb-4 ${isDarkMode ? 'text-white' : 'text-slate-900'}`}>
        React Native Base Template
      </Text>
      
      <TouchableOpacity 
        className="bg-blue-500 px-6 py-3 rounded-full active:bg-blue-600"
        onPress={toggleDarkMode}
      >
        <Text className="text-white font-semibold">Toggle Theme</Text>
      </TouchableOpacity>
      
      <StatusBar style={isDarkMode ? "light" : "dark"} />
    </View>
  );
}
```

## Important Directives
- **Zero-Comment Policy**: Do not generate extraneous comments inside the source code files. Code must be self-documenting.
- **Fail-Fast**: Ensure that if the initial `npx create-expo-app` fails (e.g. directory not empty), you handle it gracefully and inform the user.
