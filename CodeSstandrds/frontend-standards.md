# Frontend Development Standards & Guidelines

## Table of Contents
1. [Overview](#overview)
2. [Technology Stack](#technology-stack)
3. [Project Structure](#project-structure)
4. [Code Standards](#code-standards)
5. [Component Guidelines](#component-guidelines)
6. [State Management](#state-management)
7. [Styling Guidelines](#styling-guidelines)
8. [Testing Standards](#testing-standards)
9. [Git Workflow](#git-workflow)
10. [Best Practices](#best-practices)

---

## Overview

This document outlines the development standards and best practices for frontend development teams working on web (React.js) and mobile (React Native) applications.

**Key Principles:**
- Write clean, maintainable, and scalable code
- Follow consistent patterns across projects
- Prioritize performance and user experience
- Ensure code is testable and well-documented

---

## Technology Stack

### Web Applications (React.js)
- **Framework:** React 18+
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **Component Library:** shadcn/ui
- **Build Tool:** Vite or Next.js
- **State Management:** React Context API / Zustand / Redux Toolkit
- **Routing:** React Router v6+ or Next.js routing
- **HTTP Client:** Axios or Fetch API with custom hooks
- **Form Handling:** React Hook Form + Zod validation

### Mobile Applications (React Native)
- **Framework:** React Native 0.72+
- **Language:** TypeScript
- **Styling:** NativeWind (Tailwind for React Native) or styled-components
- **UI Components:** React Native Paper or NativeBase
- **Navigation:** React Navigation v6+
- **State Management:** React Context API / Zustand
- **HTTP Client:** Axios with custom hooks
- **Form Handling:** React Hook Form + Zod validation

---

## Project Structure

### React.js Web Application Structure

```
project-root/
├── public/
│   ├── assets/
│   └── favicon.ico
├── src/
│   ├── api/
│   │   ├── endpoints/
│   │   │   ├── auth.api.ts
│   │   │   └── users.api.ts
│   │   └── client.ts
│   ├── components/
│   │   ├── ui/              # shadcn components
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   └── ...
│   │   ├── common/          # Shared components
│   │   │   ├── Header.tsx
│   │   │   ├── Footer.tsx
│   │   │   └── Layout.tsx
│   │   └── features/        # Feature-specific components
│   │       ├── auth/
│   │       │   ├── LoginForm.tsx
│   │       │   └── RegisterForm.tsx
│   │       └── dashboard/
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useApi.ts
│   │   └── useLocalStorage.ts
│   ├── lib/
│   │   ├── utils.ts
│   │   └── constants.ts
│   ├── pages/               # Route pages
│   │   ├── HomePage.tsx
│   │   ├── DashboardPage.tsx
│   │   └── auth/
│   ├── store/               # State management
│   │   ├── slices/
│   │   └── index.ts
│   ├── types/
│   │   ├── api.types.ts
│   │   └── common.types.ts
│   ├── utils/
│   │   ├── validation.ts
│   │   └── helpers.ts
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css
├── .env.example
├── .eslintrc.json
├── .prettierrc
├── tailwind.config.js
├── tsconfig.json
├── package.json
└── README.md
```

### React Native Mobile Application Structure

```
project-root/
├── android/
├── ios/
├── src/
│   ├── api/
│   │   ├── endpoints/
│   │   └── client.ts
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   └── Input.tsx
│   │   └── features/
│   ├── hooks/
│   ├── navigation/
│   │   ├── AppNavigator.tsx
│   │   ├── AuthNavigator.tsx
│   │   └── types.ts
│   ├── screens/
│   │   ├── HomeScreen.tsx
│   │   ├── ProfileScreen.tsx
│   │   └── auth/
│   ├── store/
│   ├── theme/
│   │   ├── colors.ts
│   │   ├── spacing.ts
│   │   └── typography.ts
│   ├── types/
│   ├── utils/
│   ├── App.tsx
│   └── index.tsx
├── .env.example
├── .eslintrc.json
├── .prettierrc
├── package.json
├── tsconfig.json
└── README.md
```

---

## Code Standards

### 1. TypeScript Configuration

**Always use TypeScript with strict mode enabled.**

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

### 2. Naming Conventions

- **Files/Folders:** PascalCase for components (`UserProfile.tsx`), camelCase for utilities (`formatDate.ts`)
- **Components:** PascalCase (`UserCard`, `LoginForm`)
- **Functions/Variables:** camelCase (`getUserData`, `isLoading`)
- **Constants:** UPPER_SNAKE_CASE (`API_BASE_URL`, `MAX_RETRY_COUNT`)
- **Interfaces/Types:** PascalCase with descriptive names (`User`, `ApiResponse<T>`)
- **Enums:** PascalCase (`enum UserRole { Admin, User, Guest }`)

### 3. File Naming

```
✅ Good:
- UserProfile.tsx
- useAuth.ts
- api.types.ts
- Button.tsx

❌ Bad:
- userprofile.tsx
- UseAuth.ts
- API_TYPES.ts
- button.tsx
```

### 4. Import Organization

```typescript
// 1. External libraries
import React, { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

// 2. Internal modules (absolute imports)
import { Button } from '@/components/ui/button';
import { useAuth } from '@/hooks/useAuth';

// 3. Relative imports
import { formatDate } from '../utils/helpers';

// 4. Types
import type { User } from '@/types/user.types';

// 5. Styles (if applicable)
import './styles.css';
```

### 5. ESLint & Prettier Configuration

```json
// .eslintrc.json
{
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "plugin:@typescript-eslint/recommended",
    "prettier"
  ],
  "rules": {
    "react/react-in-jsx-scope": "off",
    "react/prop-types": "off",
    "@typescript-eslint/explicit-module-boundary-types": "off",
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }]
  }
}
```

```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "avoid"
}
```

---

## Component Guidelines

### 1. Functional Components with TypeScript

```typescript
// ✅ Good: Clear props interface, functional component
interface UserCardProps {
  user: User;
  onEdit?: (userId: string) => void;
  className?: string;
}

export const UserCard: React.FC<UserCardProps> = ({ user, onEdit, className }) => {
  const handleEdit = () => {
    onEdit?.(user.id);
  };

  return (
    <div className={cn('rounded-lg border p-4', className)}>
      <h3 className="text-lg font-semibold">{user.name}</h3>
      <p className="text-sm text-muted-foreground">{user.email}</p>
      {onEdit && (
        <Button onClick={handleEdit} variant="outline" size="sm">
          Edit
        </Button>
      )}
    </div>
  );
};
```

### 2. Custom Hooks

```typescript
// hooks/useApi.ts
import { useState, useEffect } from 'react';
import type { ApiResponse } from '@/types/api.types';

export function useApi<T>(endpoint: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(endpoint);
        const result: ApiResponse<T> = await response.json();
        setData(result.data);
      } catch (err) {
        setError(err as Error);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [endpoint]);

  return { data, loading, error };
}
```

### 3. Component Organization

```typescript
// Standard component structure
import React, { useState, useEffect } from 'react';

// Types at the top
interface Props {
  // ...
}

// Component definition
export const MyComponent: React.FC<Props> = ({ prop1, prop2 }) => {
  // 1. Hooks
  const [state, setState] = useState();
  
  // 2. Effects
  useEffect(() => {
    // ...
  }, []);
  
  // 3. Event handlers
  const handleClick = () => {
    // ...
  };
  
  // 4. Helper functions
  const formatData = (data: any) => {
    // ...
  };
  
  // 5. Early returns
  if (!data) return <LoadingSpinner />;
  
  // 6. Main render
  return (
    <div>
      {/* JSX */}
    </div>
  );
};
```

### 4. Props Destructuring

```typescript
// ✅ Good: Destructure props
export const UserProfile: React.FC<UserProfileProps> = ({ 
  user, 
  isEditable = false,
  onSave 
}) => {
  // Component logic
};

// ❌ Bad: Using props object
export const UserProfile: React.FC<UserProfileProps> = (props) => {
  return <div>{props.user.name}</div>;
};
```

---

## State Management

### 1. Local State (useState)

Use for component-specific state that doesn't need to be shared.

```typescript
const [isOpen, setIsOpen] = useState(false);
const [formData, setFormData] = useState<FormData>({ name: '', email: '' });
```

### 2. Context API (Small to Medium Apps)

```typescript
// contexts/AuthContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

interface AuthContextType {
  user: User | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider = ({ children }: { children: ReactNode }) => {
  const [user, setUser] = useState<User | null>(null);

  const login = async (credentials: Credentials) => {
    // Login logic
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
};
```

### 3. Zustand (Recommended for Most Apps)

```typescript
// store/authStore.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

interface AuthState {
  user: User | null;
  token: string | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  devtools(
    persist(
      (set) => ({
        user: null,
        token: null,
        login: async (credentials) => {
          const response = await authApi.login(credentials);
          set({ user: response.user, token: response.token });
        },
        logout: () => set({ user: null, token: null }),
      }),
      { name: 'auth-storage' }
    )
  )
);
```

---

## Styling Guidelines

### Web - Tailwind CSS with shadcn/ui

#### 1. Installation & Setup

```bash
npx shadcn-ui@latest init
```

#### 2. Tailwind Best Practices

```typescript
// ✅ Good: Use Tailwind utility classes
<div className="flex items-center justify-between rounded-lg border bg-card p-4 shadow-sm">
  <h2 className="text-xl font-semibold">Title</h2>
</div>

// ✅ Good: Use cn() utility for conditional classes
import { cn } from '@/lib/utils';

<Button 
  className={cn(
    'w-full',
    isLoading && 'opacity-50 cursor-not-allowed',
    variant === 'primary' && 'bg-primary text-primary-foreground'
  )}
>
  Click me
</Button>
```

#### 3. Component Composition

```typescript
// Use shadcn components as base
import { Button } from '@/components/ui/button';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';

export const DashboardCard = ({ title, children }) => (
  <Card>
    <CardHeader>
      <CardTitle>{title}</CardTitle>
    </CardHeader>
    <CardContent>{children}</CardContent>
  </Card>
);
```

### Mobile - NativeWind or Styled Components

#### Option 1: NativeWind (Recommended)

```typescript
// Setup NativeWind for Tailwind-like syntax
import { View, Text } from 'react-native';

export const ProfileCard = ({ user }) => (
  <View className="rounded-lg border border-gray-200 bg-white p-4 shadow-sm">
    <Text className="text-xl font-semibold text-gray-900">{user.name}</Text>
    <Text className="text-sm text-gray-500">{user.email}</Text>
  </View>
);
```

#### Option 2: React Native Paper

```typescript
import { Card, Title, Paragraph, Button } from 'react-native-paper';

export const ProfileCard = ({ user }) => (
  <Card>
    <Card.Content>
      <Title>{user.name}</Title>
      <Paragraph>{user.email}</Paragraph>
    </Card.Content>
    <Card.Actions>
      <Button>Edit</Button>
    </Card.Actions>
  </Card>
);
```

---

## Testing Standards

### 1. Testing Libraries

- **Unit/Component Tests:** Vitest + React Testing Library
- **E2E Tests:** Playwright or Cypress

### 2. Test File Structure

```
src/
├── components/
│   ├── UserCard.tsx
│   └── UserCard.test.tsx
```

### 3. Writing Tests

```typescript
// UserCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { UserCard } from './UserCard';

describe('UserCard', () => {
  const mockUser = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
  };

  it('renders user information correctly', () => {
    render(<UserCard user={mockUser} />);
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  it('calls onEdit when edit button is clicked', () => {
    const onEdit = vi.fn();
    render(<UserCard user={mockUser} onEdit={onEdit} />);
    
    fireEvent.click(screen.getByRole('button', { name: /edit/i }));
    expect(onEdit).toHaveBeenCalledWith('1');
  });
});
```

### 4. Test Coverage Requirements

- Aim for **80%+ code coverage**
- All critical paths must be tested
- All public APIs and hooks must have tests

---

## Git Workflow

### 1. Branch Naming

```
feature/user-authentication
bugfix/login-validation
hotfix/security-patch
chore/update-dependencies
```

### 2. Commit Messages

Follow Conventional Commits:

```
feat: add user profile page
fix: resolve login redirect issue
docs: update API documentation
style: format code with prettier
refactor: simplify authentication logic
test: add tests for user service
chore: update dependencies
```

### 3. Pull Request Process

1. Create feature branch from `develop`
2. Make changes and commit with clear messages
3. Write/update tests
4. Run linter and tests locally
5. Create PR with description of changes
6. Request code review from at least one team member
7. Address review comments
8. Merge after approval

---

## Best Practices

### 1. Performance Optimization

```typescript
// ✅ Memoize expensive calculations
const expensiveValue = useMemo(() => {
  return calculateExpensiveValue(data);
}, [data]);

// ✅ Memoize callbacks
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);

// ✅ Lazy load components
const LazyComponent = lazy(() => import('./LazyComponent'));

// ✅ Code splitting
const Dashboard = lazy(() => import('@/pages/Dashboard'));
```

### 2. Error Handling

```typescript
// API calls with proper error handling
try {
  const response = await api.getUsers();
  setUsers(response.data);
} catch (error) {
  if (error instanceof ApiError) {
    toast.error(error.message);
  } else {
    toast.error('An unexpected error occurred');
  }
  console.error('Failed to fetch users:', error);
}
```

### 3. Environment Variables

```bash
# .env.example
VITE_API_BASE_URL=https://api.example.com
VITE_APP_NAME=MyApp
```

```typescript
// Access in code
const apiUrl = import.meta.env.VITE_API_BASE_URL;
```

### 4. Accessibility

```typescript
// ✅ Good: Proper ARIA labels and semantic HTML
<button
  aria-label="Close dialog"
  onClick={handleClose}
  className="absolute right-2 top-2"
>
  <X className="h-4 w-4" />
</button>

// ✅ Good: Keyboard navigation
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => e.key === 'Enter' && handleClick()}
>
  Click me
</div>
```

### 5. Security

- Never commit sensitive data (API keys, tokens)
- Sanitize user inputs
- Use HTTPS for all API calls
- Implement proper authentication/authorization
- Keep dependencies updated

### 6. Code Review Checklist

- [ ] Code follows project structure and naming conventions
- [ ] TypeScript types are properly defined
- [ ] Components are properly tested
- [ ] No console.logs in production code
- [ ] Error handling is implemented
- [ ] Code is properly documented
- [ ] No unused imports or variables
- [ ] Accessibility standards are met

---

## Additional Resources

- [React Documentation](https://react.dev)
- [React Native Documentation](https://reactnative.dev)
- [TypeScript Documentation](https://www.typescriptlang.org/docs)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [shadcn/ui Documentation](https://ui.shadcn.com)
- [React Testing Library](https://testing-library.com/react)

---

**Last Updated:** January 2026  
**Version:** 1.0
