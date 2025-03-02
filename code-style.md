---
title: Code Style
layout: home
nav_order: 2
---

# Code Style

This document establishes consistent coding standards, architectural patterns, and best practices to ensure high-quality, maintainable code across applications.

- [Naming Conventions](#naming-conventions)
  - [Files and Folders](#files-and-folders)
  - [TypeScript](#typescript)
  - [Components and Props](#components-and-props)
- [TypeScript Best Practices](#typescript-best-practices)
  - [Use Explicit Types](#use-explicit-types)
  - [Use TypeScript Utility Types](#use-typescript-utility-types)
  - [Use Template Literal Types](#use-template-literal-types)
- [Component Structure](#component-structure)
  - [Functional Components](#functional-components)
  - [Component Organization](#component-organization)
- [Hooks Usage](#hooks-usage)
  - [Custom Hooks](#custom-hooks)
  - [Rules for Hooks](#rules-for-hooks)
- [MVVM Architecture](#mvvm-architecture)
  - [Overview](#overview)
  - [Implementation](#implementation)
    - [Model](#model)
    - [ViewModel](#viewmodel)
    - [View (Component)](#view-component)
  - [Example Project Structure](#example-project-structure)
- [File Organization](#file-organization)

## Naming Conventions

### Files and Folders

- Use **PascalCase** for components: `Button.tsx`, `UserProfile.tsx`
- Use **camelCase** for non-component files: `useAuth.ts`, `api.ts`
- Use **kebab-case** for assets: `profile-icon.png`
- Group related files in a folder:
  ```
  Button/
  ├── Button.tsx
  ├── Button.test.tsx
  ├── Button.styles.ts
  └── index.ts         # Export the component
  ```

### TypeScript

- **Interfaces**: PascalCase prefixed with `I`:
  ```typescript
  interface IUser {
    id: string;
    name: string;
    email: string;
  }
  ```
- **Types**: PascalCase:
  ```typescript
  type ButtonVariant = "primary" | "secondary" | "tertiary";
  ```
- **Enums**: PascalCase:
  ```typescript
  enum UserRole {
    Admin = "ADMIN",
    Moderator = "MODERATOR",
    User = "USER",
  }
  ```

### Components and Props

- Props: Use `Props` suffix:
  ```typescript
  interface ButtonProps {
    variant: ButtonVariant;
    label: string;
    onClick: () => void;
  }
  ```
- Event handlers: Prefix with `handle`:
  ```typescript
  const handleSubmit = () => {
    /* ... */
  };
  ```
- Event props: Prefix with `on`:
  ```typescript
  interface ButtonProps {
    onClick: () => void;
  }
  ```

## TypeScript Best Practices

### Use Explicit Types

Avoid `any` wherever possible. If needed, prefer `unknown` and then narrow the type.

```typescript
// Bad
const processData = (data: any) => {
  console.log(data.name); // Unsafe
};

// Good
const processData = (data: unknown) => {
  if (typeof data === "object" && data && "name" in data) {
    console.log(data.name);
  }
};

// Better
interface IUserData {
  name: string;
  age: number;
}

const processData = (data: IUserData) => {
  console.log(data.name); // Type-safe
};
```

### Use TypeScript Utility Types

Leverage built-in utility types:

```typescript
// Extract fields from an interface
type UserName = Pick<IUser, "firstName" | "lastName">;

// Make all fields optional
type PartialUser = Partial<IUser>;

// Make all fields required
type RequiredUser = Required<IUser>;

// Make all fields readonly
type ReadonlyUser = Readonly<IUser>;

// Exclude certain types
type NumericOrString = string | number | boolean;
type JustNumericOrString = Exclude<NumericOrString, boolean>;
```

### Use Template Literal Types

For more powerful type checking:

```typescript
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
type Endpoint = "/users" | "/posts" | "/comments";
type ValidRequest = `${HTTPMethod} ${Endpoint}`;

// ValidRequest will only accept: 'GET /users', 'POST /users', etc.
```

## Component Structure

### Functional Components

Use arrow function syntax with explicit typing:

```typescript
import React from "react";

interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
}

export const Button: React.FC<ButtonProps> = ({
  label,
  onClick,
  disabled = false,
}) => {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
};
```

### Component Organization

Follow this order inside component files:

1. Import statements
2. Type definitions and interfaces
3. Component definition
4. Helper functions and hooks
5. Export statement

```typescript
// 1. Imports
import React, { useState, useEffect } from "react";
import { Text, View } from "react-native";

// 2. Type definitions
interface UserProfileProps {
  userId: string;
  showDetails: boolean;
}

interface IUserData {
  name: string;
  email: string;
  bio: string;
}

// 3. Component
export const UserProfile: React.FC<UserProfileProps> = ({
  userId,
  showDetails,
}) => {
  // State and hooks
  const [userData, setUserData] = useState<IUserData | null>(null);
  const [loading, setLoading] = useState<boolean>(true);

  // Effects
  useEffect(() => {
    fetchUserData();
  }, [userId]);

  // Helper function
  const fetchUserData = async () => {
    setLoading(true);
    try {
      const data = await fetchUser(userId);
      setUserData(data);
    } catch (error) {
      console.error("Failed to fetch user data", error);
    } finally {
      setLoading(false);
    }
  };

  // Rendering
  if (loading) return <Text>Loading...</Text>;
  if (!userData) return <Text>User not found</Text>;

  return (
    <View>
      <Text>{userData.name}</Text>
      {showDetails && (
        <>
          <Text>{userData.email}</Text>
          <Text>{userData.bio}</Text>
        </>
      )}
    </View>
  );
};
```

## Hooks Usage

### Custom Hooks

Extract reusable logic into custom hooks:

```typescript
import { useState, useEffect } from "react";

interface UseApiOptions<T> {
  url: string;
  initialData?: T;
  autoFetch?: boolean;
}

export const useApi = <T>({
  url,
  initialData,
  autoFetch = true,
}: UseApiOptions<T>) => {
  const [data, setData] = useState<T | undefined>(initialData);
  const [loading, setLoading] = useState<boolean>(autoFetch);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = async () => {
    setLoading(true);
    setError(null);
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP error: ${response.status}`);
      }
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err : new Error(String(err)));
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    if (autoFetch) {
      fetchData();
    }
  }, [url]);

  return { data, loading, error, refetch: fetchData };
};
```

### Rules for Hooks

- Only call hooks at the top level of your components or custom hooks
- Always include all dependencies in useEffect dependency arrays
- Use ESLint with `eslint-plugin-react-hooks` to enforce these rules

```typescript
// Good
useEffect(() => {
  fetchData(userId);
}, [userId, fetchData]);

// Bad - missing dependency
useEffect(() => {
  fetchData(userId);
}, []);
```

## MVVM Architecture

### Overview

Model-View-ViewModel (MVVM) is an architectural pattern that separates UI logic from business logic. The three main components are:

- **Model**: Represents data and business logic
- **View**: The UI components (React components)
- **ViewModel**: Mediates between Model and View, handles UI logic

In our implementation, we keep these three components together in a feature folder, promoting feature-based organization rather than separating them by technical layer.

### Implementation

#### 1. Model

```typescript
// src/features/todos/TodoModel.ts
export interface ITodo {
  id: string;
  title: string;
  completed: boolean;
  createdAt: Date;
}

export class TodoModel {
  private todos: ITodo[] = [];

  getTodos(): ITodo[] {
    return [...this.todos];
  }

  addTodo(title: string): void {
    const newTodo: ITodo = {
      id: Date.now().toString(),
      title,
      completed: false,
      createdAt: new Date(),
    };
    this.todos.push(newTodo);
  }

  toggleTodo(id: string): void {
    this.todos = this.todos.map((todo) =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    );
  }

  deleteTodo(id: string): void {
    this.todos = this.todos.filter((todo) => todo.id !== id);
  }
}
```

#### 2. ViewModel

```typescript
// src/features/todos/TodoViewModel.ts
import { useState, useCallback, useEffect } from "react";
import { TodoModel, ITodo } from "./TodoModel";

export const useTodoViewModel = () => {
  const [model] = useState(() => new TodoModel());
  const [todos, setTodos] = useState<ITodo[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const refreshTodos = useCallback(() => {
    try {
      setLoading(true);
      const data = model.getTodos();
      setTodos(data);
      setError(null);
    } catch (err) {
      setError("Failed to load todos");
      console.error(err);
    } finally {
      setLoading(false);
    }
  }, [model]);

  useEffect(() => {
    refreshTodos();
  }, [refreshTodos]);

  const addTodo = useCallback(
    (title: string) => {
      try {
        model.addTodo(title);
        refreshTodos();
      } catch (err) {
        setError("Failed to add todo");
        console.error(err);
      }
    },
    [model, refreshTodos]
  );

  const toggleTodo = useCallback(
    (id: string) => {
      try {
        model.toggleTodo(id);
        refreshTodos();
      } catch (err) {
        setError("Failed to update todo");
        console.error(err);
      }
    },
    [model, refreshTodos]
  );

  const deleteTodo = useCallback(
    (id: string) => {
      try {
        model.deleteTodo(id);
        refreshTodos();
      } catch (err) {
        setError("Failed to delete todo");
        console.error(err);
      }
    },
    [model, refreshTodos]
  );

  return {
    todos,
    loading,
    error,
    addTodo,
    toggleTodo,
    deleteTodo,
  };
};
```

#### 3. View (Component)

```typescript
// src/features/todos/TodoList.tsx
import React, { useState } from "react";
import { useTodoViewModel } from "./TodoViewModel";

export const TodoList: React.FC = () => {
  const [newTodoTitle, setNewTodoTitle] = useState("");
  const { todos, loading, error, addTodo, toggleTodo, deleteTodo } =
    useTodoViewModel();

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (newTodoTitle.trim()) {
      addTodo(newTodoTitle);
      setNewTodoTitle("");
    }
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h1>Todo List</h1>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={newTodoTitle}
          onChange={(e) => setNewTodoTitle(e.target.value)}
          placeholder="Add a new todo"
        />
        <button type="submit">Add</button>
      </form>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span
              style={{
                textDecoration: todo.completed ? "line-through" : "none",
              }}
            >
              {todo.title}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
};
```

### Example Project Structure

For MVVM architecture, organize your project by features, keeping related Model, View, and ViewModel components together:

```
src/
├── features/         # Organized by domain/feature
│   ├── todos/        # Todo feature module
│   │   ├── TodoModel.ts           # Model
│   │   ├── TodoViewModel.ts       # ViewModel
│   │   ├── TodoList.tsx           # View component
│   │   ├── TodoItem.tsx           # Sub-component
│   │   ├── TodoList.test.tsx      # Tests
│   │   └── index.ts               # Exports
│   ├── users/        # User feature module
│   │   ├── UserModel.ts
│   │   ├── UserViewModel.ts
│   │   ├── UserProfile.tsx
│   │   └── index.ts
│   └── ...
├── components/       # Shared UI components
├── services/         # External services, API clients
├── utils/            # Helper functions
└── App.tsx           # Application entry point
```

This feature-based organization provides several benefits:

- Related code is grouped together, making it easier to locate and work with
- Changes to a feature affect files in a single directory
- Better code cohesion and modularity
- Easier to understand the boundaries between different parts of the application
- Simpler to add or remove entire features

## File Organization

```
src/
├── assets/             # Static files like images, fonts
├── components/         # Shared components
│   ├── common/         # Truly generic components
│   ├── forms/          # Form-specific components
│   └── layout/         # Layout components
├── features/           # Feature modules using MVVM pattern
│   ├── todos/          # Todo feature
│   ├── auth/           # Authentication feature
│   └── profile/        # User profile feature
├── hooks/              # Shared custom React hooks
├── navigation/         # (React Native) Navigation configuration
├── screens/            # (React Native) or pages/ (React.js) or app/ (Next.js)
├── services/           # API clients, utilities for data fetching
├── store/              # Global state management
│   ├── actions/        # Redux actions or equivalent
│   ├── reducers/       # Redux reducers or equivalent
│   └── selectors/      # Redux selectors or equivalent
├── utils/              # Helper functions and utilities
└── App.tsx             # Application entry point
```
