# Component Documentation

## Overview
This document provides comprehensive documentation for all UI components in the Login Application (aplikasi-login).

## Component Architecture
The application follows a modular component architecture with reusable, accessible, and well-tested components.

## Base Components

### LoginForm
The main login form component that handles user authentication.

**Props:**
```typescript
interface LoginFormProps {
  onSubmit: (credentials: LoginCredentials) => Promise<void>;
  isLoading?: boolean;
  error?: string;
  initialEmail?: string;
  showRememberMe?: boolean;
  onForgotPassword?: () => void;
  className?: string;
}

interface LoginCredentials {
  email: string;
  password: string;
  rememberMe?: boolean;
}
```

**Usage:**
```jsx
import { LoginForm } from './components/LoginForm';

function LoginPage() {
  const handleLogin = async (credentials) => {
    try {
      await authService.login(credentials);
      navigate('/home');
    } catch (error) {
      setError(error.message);
    }
  };

  return (
    <LoginForm
      onSubmit={handleLogin}
      isLoading={isAuthenticating}
      error={authError}
      showRememberMe={true}
      onForgotPassword={() => navigate('/forgot-password')}
    />
  );
}
```

**Features:**
- Email and password validation
- Loading states with disabled inputs
- Error message display
- Remember me functionality
- Forgot password link
- Accessible form labels and ARIA attributes
- Responsive design

**Validation Rules:**
- Email: Must be valid email format
- Password: Minimum 8 characters, at least one uppercase, lowercase, and number

### Button
Reusable button component with multiple variants and states.

**Props:**
```typescript
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger' | 'ghost';
  size?: 'small' | 'medium' | 'large';
  isLoading?: boolean;
  disabled?: boolean;
  type?: 'button' | 'submit' | 'reset';
  fullWidth?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
  onClick?: (event: React.MouseEvent) => void;
  children: React.ReactNode;
  className?: string;
}
```

**Usage:**
```jsx
import { Button } from './components/Button';

// Primary button
<Button variant="primary" size="large" fullWidth>
  Login
</Button>

// Loading state
<Button variant="primary" isLoading>
  Signing in...
</Button>

// With icons
<Button leftIcon={<GoogleIcon />} variant="secondary">
  Sign in with Google
</Button>
```

### Input
Form input component with validation and accessibility features.

**Props:**
```typescript
interface InputProps {
  type?: 'text' | 'email' | 'password' | 'tel' | 'url';
  label: string;
  placeholder?: string;
  value: string;
  onChange: (value: string) => void;
  onBlur?: () => void;
  error?: string;
  required?: boolean;
  disabled?: boolean;
  autoComplete?: string;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
  helperText?: string;
  className?: string;
}
```

**Usage:**
```jsx
import { Input } from './components/Input';

<Input
  type="email"
  label="Email Address"
  placeholder="Enter your email"
  value={email}
  onChange={setEmail}
  error={emailError}
  required
  autoComplete="email"
  leftIcon={<EmailIcon />}
/>
```

### Alert
Component for displaying various types of messages and notifications.

**Props:**
```typescript
interface AlertProps {
  variant: 'success' | 'error' | 'warning' | 'info';
  title?: string;
  message: string;
  dismissible?: boolean;
  onDismiss?: () => void;
  actions?: Array<{
    label: string;
    onClick: () => void;
    variant?: 'primary' | 'secondary';
  }>;
  className?: string;
}
```

**Usage:**
```jsx
import { Alert } from './components/Alert';

// Error alert
<Alert
  variant="error"
  title="Login Failed"
  message="Email atau password tidak valid"
  dismissible
  onDismiss={() => setError(null)}
/>

// Success alert with actions
<Alert
  variant="success"
  message="Password reset email sent successfully"
  actions={[
    { label: 'Resend Email', onClick: resendEmail },
    { label: 'Back to Login', onClick: () => navigate('/login') }
  ]}
/>
```

## Layout Components

### Header
Application header with navigation and user menu.

**Props:**
```typescript
interface HeaderProps {
  user?: User;
  onLogout?: () => void;
  showLogo?: boolean;
  navigation?: NavigationItem[];
  className?: string;
}

interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
}

interface NavigationItem {
  label: string;
  href: string;
  active?: boolean;
  icon?: React.ReactNode;
}
```

**Usage:**
```jsx
import { Header } from './components/Header';

<Header
  user={currentUser}
  onLogout={handleLogout}
  showLogo={true}
  navigation={[
    { label: 'Home', href: '/home', active: true },
    { label: 'Profile', href: '/profile' },
    { label: 'Settings', href: '/settings' }
  ]}
/>
```

### Layout
Main layout wrapper component.

**Props:**
```typescript
interface LayoutProps {
  children: React.ReactNode;
  sidebar?: React.ReactNode;
  header?: React.ReactNode;
  footer?: React.ReactNode;
  className?: string;
}
```

**Usage:**
```jsx
import { Layout } from './components/Layout';

<Layout
  header={<Header user={user} onLogout={logout} />}
  sidebar={<Sidebar />}
>
  <main>
    {children}
  </main>
</Layout>
```

## Form Components

### FormField
Wrapper component for form fields with consistent styling and validation.

**Props:**
```typescript
interface FormFieldProps {
  children: React.ReactNode;
  label?: string;
  error?: string;
  required?: boolean;
  helperText?: string;
  className?: string;
}
```

### Checkbox
Checkbox input component with custom styling.

**Props:**
```typescript
interface CheckboxProps {
  label: string;
  checked: boolean;
  onChange: (checked: boolean) => void;
  disabled?: boolean;
  error?: string;
  className?: string;
}
```

**Usage:**
```jsx
<Checkbox
  label="Remember me"
  checked={rememberMe}
  onChange={setRememberMe}
/>
```

## Higher-Order Components

### withAuth
HOC for protecting routes that require authentication.

**Usage:**
```jsx
import { withAuth } from './hoc/withAuth';

const ProtectedComponent = withAuth(HomeComponent);

// Or with options
const ProtectedAdmin = withAuth(AdminComponent, {
  requiredRole: 'admin',
  redirectTo: '/unauthorized'
});
```

### withLoading
HOC for adding loading states to components.

**Usage:**
```jsx
import { withLoading } from './hoc/withLoading';

const LoadingAwareComponent = withLoading(MyComponent);

<LoadingAwareComponent isLoading={true} />
```

## Utility Components

### ErrorBoundary
Component for catching and handling React errors.

**Props:**
```typescript
interface ErrorBoundaryProps {
  children: React.ReactNode;
  fallback?: React.ComponentType<{ error: Error; retry: () => void }>;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}
```

**Usage:**
```jsx
import { ErrorBoundary } from './components/ErrorBoundary';

<ErrorBoundary
  fallback={CustomErrorComponent}
  onError={(error) => logError(error)}
>
  <App />
</ErrorBoundary>
```

### PrivateRoute
Component for protecting routes that require authentication.

**Props:**
```typescript
interface PrivateRouteProps {
  children: React.ReactNode;
  requiredRole?: string;
  fallback?: React.ReactNode;
  redirectTo?: string;
}
```

**Usage:**
```jsx
import { PrivateRoute } from './components/PrivateRoute';

<PrivateRoute requiredRole="user" redirectTo="/login">
  <HomePage />
</PrivateRoute>
```

## Component Testing

### Testing Guidelines
All components should include comprehensive tests covering:

1. **Rendering Tests**: Component renders without crashing
2. **Props Tests**: All props are handled correctly
3. **Interaction Tests**: User interactions work as expected
4. **Accessibility Tests**: Components meet ARIA standards
5. **Error State Tests**: Error conditions are handled gracefully

### Example Test
```javascript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('should call onSubmit with correct credentials', async () => {
    const mockSubmit = jest.fn();
    
    render(<LoginForm onSubmit={mockSubmit} />);
    
    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'test@example.com' }
    });
    
    fireEvent.change(screen.getByLabelText(/password/i), {
      target: { value: 'password123' }
    });
    
    fireEvent.click(screen.getByRole('button', { name: /login/i }));
    
    await waitFor(() => {
      expect(mockSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123'
      });
    });
  });
});
```

## Styling Guidelines

### CSS-in-JS with Styled Components
```jsx
import styled from 'styled-components';

const StyledButton = styled.button`
  padding: ${props => props.size === 'large' ? '12px 24px' : '8px 16px'};
  background-color: ${props => props.variant === 'primary' ? '#007bff' : '#6c757d'};
  border: none;
  border-radius: 4px;
  color: white;
  cursor: pointer;
  
  &:hover {
    opacity: 0.9;
  }
  
  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
`;
```

### CSS Modules Alternative
```css
/* Button.module.css */
.button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
}

.button--large {
  padding: 12px 24px;
}

.button--primary {
  background-color: #007bff;
  color: white;
}
```

## Accessibility Guidelines

### ARIA Labels and Roles
- Use semantic HTML elements when possible
- Add appropriate ARIA labels and descriptions
- Ensure keyboard navigation works correctly
- Maintain proper focus management
- Use sufficient color contrast ratios

### Example Accessible Component
```jsx
function AccessibleButton({ isLoading, children, ...props }) {
  return (
    <button
      {...props}
      aria-label={isLoading ? 'Loading...' : props['aria-label']}
      aria-disabled={isLoading}
      disabled={isLoading}
    >
      {isLoading ? (
        <>
          <span aria-hidden="true">⏳</span>
          <span className="sr-only">Loading...</span>
        </>
      ) : (
        children
      )}
    </button>
  );
}
```