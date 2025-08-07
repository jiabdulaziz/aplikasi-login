# Function Documentation

## Overview
This document provides comprehensive documentation for all utility functions, services, hooks, and business logic in the Login Application (aplikasi-login).

## Authentication Service

### authService.login()
Authenticates a user with email and password credentials.

**Signature:**
```typescript
function login(credentials: LoginCredentials): Promise<AuthResponse>

interface LoginCredentials {
  email: string;
  password: string;
  rememberMe?: boolean;
}

interface AuthResponse {
  token: string;
  user: User;
  expiresIn: number;
}
```

**Parameters:**
- `credentials.email` (string, required) - User's email address
- `credentials.password` (string, required) - User's password
- `credentials.rememberMe` (boolean, optional) - Whether to persist session

**Returns:**
Promise that resolves to `AuthResponse` object containing token, user data, and expiration time.

**Throws:**
- `ValidationError` - When email or password format is invalid
- `AuthenticationError` - When credentials are incorrect
- `NetworkError` - When request fails

**Example:**
```javascript
import { authService } from './services/authService';

try {
  const response = await authService.login({
    email: 'user@example.com',
    password: 'securePassword123',
    rememberMe: true
  });
  
  console.log('Login successful:', response.user);
  localStorage.setItem('token', response.token);
} catch (error) {
  if (error instanceof AuthenticationError) {
    console.error('Invalid credentials');
  } else {
    console.error('Login failed:', error.message);
  }
}
```

### authService.logout()
Logs out the current user and cleans up session data.

**Signature:**
```typescript
function logout(): Promise<void>
```

**Returns:**
Promise that resolves when logout is complete.

**Example:**
```javascript
try {
  await authService.logout();
  localStorage.removeItem('token');
  window.location.href = '/login';
} catch (error) {
  console.error('Logout failed:', error);
}
```

### authService.refreshToken()
Refreshes the authentication token before it expires.

**Signature:**
```typescript
function refreshToken(): Promise<string>
```

**Returns:**
Promise that resolves to new JWT token string.

**Example:**
```javascript
// Automatically refresh token when it's about to expire
setInterval(async () => {
  try {
    const newToken = await authService.refreshToken();
    localStorage.setItem('token', newToken);
  } catch (error) {
    // Token refresh failed, redirect to login
    window.location.href = '/login';
  }
}, 50 * 60 * 1000); // Refresh every 50 minutes
```

### authService.getCurrentUser()
Retrieves the current authenticated user's information.

**Signature:**
```typescript
function getCurrentUser(): Promise<User | null>
```

**Returns:**
Promise that resolves to `User` object or `null` if not authenticated.

**Example:**
```javascript
const user = await authService.getCurrentUser();
if (user) {
  console.log('Current user:', user.name);
} else {
  console.log('No user logged in');
}
```

## Validation Functions

### validateEmail()
Validates email address format according to RFC 5322 specification.

**Signature:**
```typescript
function validateEmail(email: string): ValidationResult

interface ValidationResult {
  isValid: boolean;
  error?: string;
}
```

**Parameters:**
- `email` (string, required) - Email address to validate

**Returns:**
`ValidationResult` object with validation status and error message.

**Example:**
```javascript
import { validateEmail } from './utils/validation';

const result = validateEmail('user@example.com');
if (result.isValid) {
  console.log('Valid email');
} else {
  console.error('Invalid email:', result.error);
}
```

### validatePassword()
Validates password strength and format requirements.

**Signature:**
```typescript
function validatePassword(password: string): PasswordValidationResult

interface PasswordValidationResult {
  isValid: boolean;
  score: number; // 0-4 strength score
  errors: string[];
  suggestions: string[];
}
```

**Parameters:**
- `password` (string, required) - Password to validate

**Requirements:**
- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- At least one special character (optional but recommended)

**Example:**
```javascript
import { validatePassword } from './utils/validation';

const result = validatePassword('MyPassword123!');
console.log('Password strength:', result.score);
console.log('Errors:', result.errors);
console.log('Suggestions:', result.suggestions);
```

### sanitizeInput()
Sanitizes user input to prevent XSS attacks and clean data.

**Signature:**
```typescript
function sanitizeInput(input: string, options?: SanitizeOptions): string

interface SanitizeOptions {
  allowHtml?: boolean;
  maxLength?: number;
  trimWhitespace?: boolean;
}
```

**Parameters:**
- `input` (string, required) - Input string to sanitize
- `options` (object, optional) - Sanitization options

**Example:**
```javascript
import { sanitizeInput } from './utils/validation';

const cleanInput = sanitizeInput('<script>alert("xss")</script>Hello', {
  allowHtml: false,
  maxLength: 100,
  trimWhitespace: true
});
console.log(cleanInput); // "Hello"
```

## Storage Utilities

### secureStorage.set()
Securely stores data with encryption and automatic expiration.

**Signature:**
```typescript
function set(key: string, value: any, options?: StorageOptions): void

interface StorageOptions {
  expiresIn?: number; // milliseconds
  encrypt?: boolean;
  persistent?: boolean;
}
```

**Parameters:**
- `key` (string, required) - Storage key
- `value` (any, required) - Value to store
- `options` (object, optional) - Storage configuration

**Example:**
```javascript
import { secureStorage } from './utils/storage';

// Store with 1 hour expiration
secureStorage.set('userToken', token, {
  expiresIn: 60 * 60 * 1000,
  encrypt: true,
  persistent: true
});
```

### secureStorage.get()
Retrieves and decrypts stored data.

**Signature:**
```typescript
function get<T>(key: string): T | null
```

**Parameters:**
- `key` (string, required) - Storage key

**Returns:**
Stored value or `null` if not found or expired.

**Example:**
```javascript
const token = secureStorage.get<string>('userToken');
if (token) {
  console.log('Token retrieved:', token);
} else {
  console.log('Token not found or expired');
}
```

## HTTP Client Functions

### apiClient.request()
Makes authenticated HTTP requests with automatic retry and error handling.

**Signature:**
```typescript
function request<T>(config: RequestConfig): Promise<ApiResponse<T>>

interface RequestConfig {
  method: 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';
  url: string;
  data?: any;
  params?: Record<string, any>;
  headers?: Record<string, string>;
  timeout?: number;
  retries?: number;
}

interface ApiResponse<T> {
  data: T;
  status: number;
  headers: Record<string, string>;
}
```

**Parameters:**
- `config` (object, required) - Request configuration

**Returns:**
Promise that resolves to `ApiResponse` with typed data.

**Example:**
```javascript
import { apiClient } from './utils/apiClient';

try {
  const response = await apiClient.request({
    method: 'POST',
    url: '/auth/login',
    data: { email, password },
    timeout: 10000,
    retries: 3
  });
  
  console.log('Response:', response.data);
} catch (error) {
  console.error('Request failed:', error);
}
```

### apiClient.get()
Convenience method for GET requests.

**Signature:**
```typescript
function get<T>(url: string, params?: Record<string, any>): Promise<T>
```

**Example:**
```javascript
const user = await apiClient.get<User>('/users/profile');
const users = await apiClient.get<User[]>('/users', { page: 1, limit: 10 });
```

### apiClient.post()
Convenience method for POST requests.

**Signature:**
```typescript
function post<T>(url: string, data?: any): Promise<T>
```

**Example:**
```javascript
const result = await apiClient.post('/auth/login', { email, password });
```

## React Hooks

### useAuth()
Custom hook for managing authentication state.

**Signature:**
```typescript
function useAuth(): AuthHookReturn

interface AuthHookReturn {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => Promise<void>;
  refreshUser: () => Promise<void>;
}
```

**Returns:**
Object with authentication state and methods.

**Example:**
```javascript
import { useAuth } from './hooks/useAuth';

function LoginPage() {
  const { login, isLoading, user, isAuthenticated } = useAuth();
  
  const handleSubmit = async (credentials) => {
    try {
      await login(credentials);
      // User will be automatically updated
    } catch (error) {
      setError(error.message);
    }
  };
  
  if (isAuthenticated) {
    return <Navigate to="/home" />;
  }
  
  return <LoginForm onSubmit={handleSubmit} isLoading={isLoading} />;
}
```

### useForm()
Custom hook for form state management and validation.

**Signature:**
```typescript
function useForm<T>(options: FormOptions<T>): FormHookReturn<T>

interface FormOptions<T> {
  initialValues: T;
  validationSchema?: ValidationSchema<T>;
  onSubmit: (values: T) => Promise<void> | void;
}

interface FormHookReturn<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
  isSubmitting: boolean;
  handleChange: (field: keyof T) => (value: any) => void;
  handleBlur: (field: keyof T) => () => void;
  handleSubmit: (e: React.FormEvent) => void;
  resetForm: () => void;
  setFieldValue: (field: keyof T, value: any) => void;
  setFieldError: (field: keyof T, error: string) => void;
}
```

**Example:**
```javascript
import { useForm } from './hooks/useForm';
import * as yup from 'yup';

const validationSchema = yup.object({
  email: yup.string().email('Invalid email').required('Required'),
  password: yup.string().min(8, 'Too short').required('Required')
});

function LoginForm() {
  const {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit
  } = useForm({
    initialValues: { email: '', password: '' },
    validationSchema,
    onSubmit: async (values) => {
      await authService.login(values);
    }
  });
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={values.email}
        onChange={handleChange('email')}
        onBlur={handleBlur('email')}
      />
      {touched.email && errors.email && <span>{errors.email}</span>}
      
      <input
        type="password"
        value={values.password}
        onChange={handleChange('password')}
        onBlur={handleBlur('password')}
      />
      {touched.password && errors.password && <span>{errors.password}</span>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

### useLocalStorage()
Custom hook for managing localStorage with React state synchronization.

**Signature:**
```typescript
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void]
```

**Parameters:**
- `key` (string, required) - localStorage key
- `initialValue` (T, required) - Default value if key doesn't exist

**Returns:**
Array with current value and setter function.

**Example:**
```javascript
import { useLocalStorage } from './hooks/useLocalStorage';

function App() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [rememberMe, setRememberMe] = useLocalStorage('rememberMe', false);
  
  return (
    <div className={`app theme-${theme}`}>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </div>
  );
}
```

## Error Handling Functions

### createError()
Creates standardized error objects with additional context.

**Signature:**
```typescript
function createError(message: string, code: string, status?: number): AppError

class AppError extends Error {
  code: string;
  status?: number;
  timestamp: Date;
  stack?: string;
}
```

**Example:**
```javascript
import { createError } from './utils/errors';

// In service function
if (!user) {
  throw createError('User not found', 'USER_NOT_FOUND', 404);
}
```

### handleApiError()
Processes API errors and converts them to user-friendly messages.

**Signature:**
```typescript
function handleApiError(error: any): UserFriendlyError

interface UserFriendlyError {
  message: string;
  code: string;
  isRetryable: boolean;
  shouldLogout: boolean;
}
```

**Example:**
```javascript
import { handleApiError } from './utils/errors';

try {
  await apiClient.post('/data');
} catch (error) {
  const userError = handleApiError(error);
  
  if (userError.shouldLogout) {
    await authService.logout();
  }
  
  if (userError.isRetryable) {
    // Show retry button
  }
  
  setErrorMessage(userError.message);
}
```

## Date and Time Utilities

### formatDate()
Formats dates according to locale and timezone preferences.

**Signature:**
```typescript
function formatDate(date: Date | string, format?: DateFormat, locale?: string): string

type DateFormat = 'short' | 'medium' | 'long' | 'full' | string;
```

**Example:**
```javascript
import { formatDate } from './utils/date';

const date = new Date();
console.log(formatDate(date, 'short')); // "1/15/24"
console.log(formatDate(date, 'medium')); // "Jan 15, 2024"
console.log(formatDate(date, 'DD/MM/YYYY')); // "15/01/2024"
```

### isTokenExpired()
Checks if a JWT token is expired or about to expire.

**Signature:**
```typescript
function isTokenExpired(token: string, bufferMinutes?: number): boolean
```

**Parameters:**
- `token` (string, required) - JWT token to check
- `bufferMinutes` (number, optional) - Minutes before expiration to consider as expired (default: 5)

**Example:**
```javascript
import { isTokenExpired } from './utils/auth';

const token = localStorage.getItem('token');
if (token && isTokenExpired(token, 10)) {
  // Token expires in less than 10 minutes, refresh it
  await authService.refreshToken();
}
```

## Performance Utilities

### debounce()
Creates a debounced function that delays execution until after wait milliseconds.

**Signature:**
```typescript
function debounce<T extends (...args: any[]) => any>(
  func: T,
  wait: number,
  immediate?: boolean
): (...args: Parameters<T>) => void
```

**Example:**
```javascript
import { debounce } from './utils/performance';

const debouncedSearch = debounce(async (query) => {
  const results = await searchAPI(query);
  setSearchResults(results);
}, 300);

// Usage in component
<input
  onChange={(e) => debouncedSearch(e.target.value)}
  placeholder="Search..."
/>
```

### throttle()
Creates a throttled function that only executes at most once per wait milliseconds.

**Signature:**
```typescript
function throttle<T extends (...args: any[]) => any>(
  func: T,
  wait: number
): (...args: Parameters<T>) => void
```

**Example:**
```javascript
import { throttle } from './utils/performance';

const throttledScroll = throttle(() => {
  console.log('Scroll event');
}, 100);

window.addEventListener('scroll', throttledScroll);
```

## Testing Utilities

### mockAuthService()
Creates a mock implementation of the auth service for testing.

**Signature:**
```typescript
function mockAuthService(overrides?: Partial<AuthService>): AuthService
```

**Example:**
```javascript
import { mockAuthService } from './utils/testing';

describe('LoginForm', () => {
  it('should handle login success', async () => {
    const mockAuth = mockAuthService({
      login: jest.fn().mockResolvedValue({
        token: 'fake-token',
        user: { id: '1', email: 'test@example.com' }
      })
    });
    
    // Test implementation
  });
});
```

### createTestWrapper()
Creates a test wrapper with providers for testing React components.

**Signature:**
```typescript
function createTestWrapper(options?: TestWrapperOptions): React.ComponentType

interface TestWrapperOptions {
  initialAuth?: User | null;
  theme?: 'light' | 'dark';
  router?: boolean;
}
```

**Example:**
```javascript
import { render } from '@testing-library/react';
import { createTestWrapper } from './utils/testing';

const TestWrapper = createTestWrapper({
  initialAuth: mockUser,
  router: true
});

const { getByText } = render(<LoginForm />, { wrapper: TestWrapper });
```