---
title: Testing
layout: home
---

## Testing

- **Write Tests:** Ensure the reliability of the codebase by writing tests for components, hooks, and business logic.
  - **Unit Tests:** Use [Jest](https://jestjs.io/) for testing individual components and functions.
  - **Integration Tests:** Test interactions between components and state management.
  - **End-to-End Tests:** Use [Detox](https://github.com/wix/Detox) for testing the app's user flows on real devices.
- **Test Coverage:** Aim for high test coverage, especially for critical paths and reusable components, to facilitate refactoring and prevent regressions.

## Testing Standards

### Unit Tests

Use Jest with React Testing Library:

```typescript
import { render, screen, fireEvent } from "@testing-library/react";
import { Button } from "./Button";

describe("Button", () => {
  test("renders with correct label", () => {
    render(<Button label="Click me" onClick={() => {}} />);
    expect(screen.getByText("Click me")).toBeInTheDocument();
  });

  test("calls onClick when clicked", () => {
    const handleClick = jest.fn();
    render(<Button label="Click me" onClick={handleClick} />);
    fireEvent.click(screen.getByText("Click me"));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

### Component Testing

Test MVVM components:

```typescript
import { render, screen, fireEvent } from "@testing-library/react";
import { TodoList } from "./TodoList";
import { useTodoViewModel } from "./TodoViewModel";

// Mock the ViewModel
jest.mock("./TodoViewModel");

describe("TodoList", () => {
  const mockViewModel = {
    todos: [
      { id: "1", title: "Test Todo", completed: false, createdAt: new Date() },
    ],
    loading: false,
    error: null,
    addTodo: jest.fn(),
    toggleTodo: jest.fn(),
    deleteTodo: jest.fn(),
  };

  beforeEach(() => {
    (useTodoViewModel as jest.Mock).mockReturnValue(mockViewModel);
  });

  test("renders todos from ViewModel", () => {
    render(<TodoList />);
    expect(screen.getByText("Test Todo")).toBeInTheDocument();
  });

  test("calls addTodo when form is submitted", () => {
    render(<TodoList />);
    const input = screen.getByPlaceholderText("Add a new todo");
    const form = screen.getByRole("form");

    fireEvent.change(input, { target: { value: "New Todo" } });
    fireEvent.submit(form);

    expect(mockViewModel.addTodo).toHaveBeenCalledWith("New Todo");
  });
});
```
