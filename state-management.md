---
title: State Management
layout: home
---

## State Management

- **Choose the Right Tool:** Depending on the complexity of the app, select an appropriate state management solution.
  - **Simple Apps:** Use the [Context API](https://react.dev/learn/passing-data-deeply-with-context) with `useReducer` for managing global state.
  - **Complex Apps:** Consider [Redux](https://redux.js.org/) or [MobX](https://mobx.js.org/) for more advanced state management needs.
- **Optimize State Updates:** Avoid unnecessary re-renders by using memoization techniques and keeping state as local as possible. Use tools like `useSelector` (Redux) or `observer` (MobX) to optimize performance.

### Local State

For component-specific state, use React's `useState` hook:

```typescript
const [count, setCount] = useState<number>(0);
```

### Context API

For shared state across components without prop drilling:

```typescript
// ThemeContext.tsx
import React, { createContext, useContext, useState } from "react";

type Theme = "light" | "dark";

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider: React.FC<React.PropsWithChildren<{}>> = ({
  children,
}) => {
  const [theme, setTheme] = useState<Theme>("light");

  const toggleTheme = () => {
    setTheme((prevTheme) => (prevTheme === "light" ? "dark" : "light"));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error("useTheme must be used within a ThemeProvider");
  }
  return context;
};
```

### Global State

For more complex applications, consider:

- Redux with TypeScript
- MobX
- Zustand (simpler alternative to Redux)
- Recoil (Facebook's experimental state management library)
