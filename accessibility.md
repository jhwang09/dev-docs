---
title: Accessibility
layout: home
---

## Accessibility

### Semantic HTML

Use the right HTML elements for the right job:

```typescript
// Bad
<div onClick={handleClick}>Click me</div>

// Good
<button onClick={handleClick}>Click me</button>
```

### ARIA Attributes

Add ARIA attributes when needed:

```typescript
<button aria-label="Close dialog" aria-expanded={isOpen} onClick={handleClose}>
  <span className="icon">×</span>
</button>
```

### Keyboard Navigation

Ensure all interactive elements are keyboard accessible:

```typescript
const handleKeyDown = (e: React.KeyboardEvent) => {
  if (e.key === "Enter" || e.key === " ") {
    handleAction();
  }
};

return (
  <div
    role="button"
    tabIndex={0}
    onClick={handleAction}
    onKeyDown={handleKeyDown}
  >
    Click or press Enter
  </div>
);
```
