# Usage Examples and Code Samples

## Overview
This document provides comprehensive usage examples and code samples for implementing the Login Application (aplikasi-login) features.

## Complete Login Flow Example

### Basic Login Implementation
```javascript
// src/pages/LoginPage.jsx
import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { LoginForm } from '../components/LoginForm';
import { Alert } from '../components/Alert';
import { authService } from '../services/authService';
import { useAuth } from '../hooks/useAuth';

function LoginPage() {
  const [error, setError] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const navigate = useNavigate();
  const { login } = useAuth();

  const handleLogin = async (credentials) => {
    setIsLoading(true);
    setError('');
    
    try {
      await login(credentials);
      navigate('/home');
    } catch (error) {
      setError(error.message);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="login-page">
      <div className="login-container">
        <h1>Masuk ke Aplikasi</h1>
        
        {error && (
          <Alert
            variant="error"
            message={error}
            dismissible
            onDismiss={() => setError('')}
          />
        )}
        
        <LoginForm
          onSubmit={handleLogin}
          isLoading={isLoading}
          showRememberMe={true}
          onForgotPassword={() => navigate('/forgot-password')}
        />
      </div>
    </div>
  );
}

export default LoginPage;
```

### Advanced Login with Validation
```javascript
// src/components/LoginForm.jsx
import React from 'react';
import { useForm } from '../hooks/useForm';
import { Input } from './Input';
import { Button } from './Button';
import { Checkbox } from './Checkbox';
import { validateEmail, validatePassword } from '../utils/validation';
import * as yup from 'yup';

const validationSchema = yup.object({
  email: yup
    .string()
    .email('Format email tidak valid')
    .required('Email wajib diisi'),
  password: yup
    .string()
    .min(8, 'Password minimal 8 karakter')
    .required('Password wajib diisi'),
  rememberMe: yup.boolean()
});

function LoginForm({ onSubmit, isLoading, error, showRememberMe = false }) {
  const {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    setFieldError
  } = useForm({
    initialValues: {
      email: '',
      password: '',
      rememberMe: false
    },
    validationSchema,
    onSubmit: async (values) => {
      try {
        await onSubmit(values);
      } catch (error) {
        if (error.field) {
          setFieldError(error.field, error.message);
        } else {
          throw error;
        }
      }
    }
  });

  return (
    <form onSubmit={handleSubmit} className="login-form">
      <Input
        type="email"
        label="Email"
        placeholder="Masukkan email Anda"
        value={values.email}
        onChange={handleChange('email')}
        onBlur={handleBlur('email')}
        error={touched.email ? errors.email : ''}
        required
        autoComplete="email"
        leftIcon={<EmailIcon />}
      />

      <Input
        type="password"
        label="Password"
        placeholder="Masukkan password Anda"
        value={values.password}
        onChange={handleChange('password')}
        onBlur={handleBlur('password')}
        error={touched.password ? errors.password : ''}
        required
        autoComplete="current-password"
        leftIcon={<LockIcon />}
      />

      {showRememberMe && (
        <Checkbox
          label="Ingat saya"
          checked={values.rememberMe}
          onChange={handleChange('rememberMe')}
        />
      )}

      <Button
        type="submit"
        variant="primary"
        size="large"
        fullWidth
        isLoading={isSubmitting || isLoading}
      >
        {isSubmitting || isLoading ? 'Masuk...' : 'Masuk'}
      </Button>
    </form>
  );
}

export default LoginForm;
```

## Authentication Hook Implementation

### useAuth Hook
```javascript
// src/hooks/useAuth.js
import { useState, useEffect, useContext, createContext } from 'react';
import { authService } from '../services/authService';
import { secureStorage } from '../utils/storage';

const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  // Check for existing session on mount
  useEffect(() => {
    const initializeAuth = async () => {
      try {
        const token = secureStorage.get('authToken');
        if (token) {
          const user = await authService.getCurrentUser();
          if (user) {
            setUser(user);
            setIsAuthenticated(true);
          }
        }
      } catch (error) {
        console.error('Auth initialization failed:', error);
        secureStorage.remove('authToken');
      } finally {
        setIsLoading(false);
      }
    };

    initializeAuth();
  }, []);

  const login = async (credentials) => {
    setIsLoading(true);
    try {
      const response = await authService.login(credentials);
      
      // Store token securely
      secureStorage.set('authToken', response.token, {
        expiresIn: response.expiresIn * 1000,
        encrypt: true,
        persistent: credentials.rememberMe
      });
      
      setUser(response.user);
      setIsAuthenticated(true);
      
      return response;
    } finally {
      setIsLoading(false);
    }
  };

  const logout = async () => {
    try {
      await authService.logout();
    } catch (error) {
      console.error('Logout failed:', error);
    } finally {
      secureStorage.remove('authToken');
      setUser(null);
      setIsAuthenticated(false);
    }
  };

  const refreshUser = async () => {
    if (isAuthenticated) {
      try {
        const updatedUser = await authService.getCurrentUser();
        setUser(updatedUser);
      } catch (error) {
        console.error('Failed to refresh user:', error);
        await logout();
      }
    }
  };

  const value = {
    user,
    isAuthenticated,
    isLoading,
    login,
    logout,
    refreshUser
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

## Route Protection Examples

### Private Route Component
```javascript
// src/components/PrivateRoute.jsx
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';
import { LoadingSpinner } from './LoadingSpinner';

function PrivateRoute({ children, requiredRole = null }) {
  const { isAuthenticated, isLoading, user } = useAuth();
  const location = useLocation();

  if (isLoading) {
    return <LoadingSpinner />;
  }

  if (!isAuthenticated) {
    return (
      <Navigate 
        to="/login" 
        state={{ from: location }} 
        replace 
      />
    );
  }

  if (requiredRole && user?.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />;
  }

  return children;
}

export default PrivateRoute;
```

### App Router Setup
```javascript
// src/App.jsx
import React from 'react';
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
import { AuthProvider } from './hooks/useAuth';
import PrivateRoute from './components/PrivateRoute';
import LoginPage from './pages/LoginPage';
import HomePage from './pages/HomePage';
import ProfilePage from './pages/ProfilePage';
import UnauthorizedPage from './pages/UnauthorizedPage';

function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          {/* Public routes */}
          <Route path="/login" element={<LoginPage />} />
          <Route path="/unauthorized" element={<UnauthorizedPage />} />
          
          {/* Protected routes */}
          <Route 
            path="/home" 
            element={
              <PrivateRoute>
                <HomePage />
              </PrivateRoute>
            } 
          />
          
          <Route 
            path="/profile" 
            element={
              <PrivateRoute>
                <ProfilePage />
              </PrivateRoute>
            } 
          />
          
          {/* Admin only routes */}
          <Route 
            path="/admin" 
            element={
              <PrivateRoute requiredRole="admin">
                <AdminPage />
              </PrivateRoute>
            } 
          />
          
          {/* Redirects */}
          <Route path="/" element={<Navigate to="/home" replace />} />
          <Route path="*" element={<Navigate to="/home" replace />} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}

export default App;
```

## API Service Implementation

### Auth Service
```javascript
// src/services/authService.js
import { apiClient } from '../utils/apiClient';
import { createError } from '../utils/errors';

class AuthService {
  async login(credentials) {
    try {
      const response = await apiClient.post('/auth/login', credentials);
      return response.data;
    } catch (error) {
      if (error.status === 401) {
        throw createError('Email atau password tidak valid', 'INVALID_CREDENTIALS');
      }
      throw error;
    }
  }

  async logout() {
    try {
      await apiClient.post('/auth/logout');
    } catch (error) {
      // Even if logout fails on server, we still want to clear local state
      console.warn('Server logout failed:', error);
    }
  }

  async refreshToken() {
    try {
      const response = await apiClient.post('/auth/refresh');
      return response.data.token;
    } catch (error) {
      throw createError('Token refresh failed', 'TOKEN_REFRESH_FAILED');
    }
  }

  async getCurrentUser() {
    try {
      const response = await apiClient.get('/users/profile');
      return response.data;
    } catch (error) {
      if (error.status === 401) {
        return null;
      }
      throw error;
    }
  }

  async updateProfile(profileData) {
    try {
      const response = await apiClient.put('/users/profile', profileData);
      return response.data;
    } catch (error) {
      throw error;
    }
  }
}

export const authService = new AuthService();
```

### API Client with Interceptors
```javascript
// src/utils/apiClient.js
import axios from 'axios';
import { secureStorage } from './storage';
import { createError } from './errors';

const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:3001/api/v1';

const apiClient = axios.create({
  baseURL: API_BASE_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor to add auth token
apiClient.interceptors.request.use(
  (config) => {
    const token = secureStorage.get('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor for error handling
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    // Handle token expiration
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        const newToken = await refreshToken();
        secureStorage.set('authToken', newToken);
        originalRequest.headers.Authorization = `Bearer ${newToken}`;
        return apiClient(originalRequest);
      } catch (refreshError) {
        // Refresh failed, redirect to login
        secureStorage.remove('authToken');
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }

    // Transform error to our standard format
    const transformedError = createError(
      error.response?.data?.message || error.message,
      error.response?.data?.code || 'NETWORK_ERROR',
      error.response?.status
    );

    return Promise.reject(transformedError);
  }
);

const refreshToken = async () => {
  const response = await axios.post(`${API_BASE_URL}/auth/refresh`, {}, {
    headers: {
      Authorization: `Bearer ${secureStorage.get('authToken')}`
    }
  });
  return response.data.token;
};

export { apiClient };
```

## Form Validation Examples

### Custom Validation Hook
```javascript
// src/hooks/useForm.js
import { useState, useCallback } from 'react';

export function useForm({ initialValues, validationSchema, onSubmit }) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const validateField = useCallback(async (field, value) => {
    if (!validationSchema) return '';

    try {
      await validationSchema.validateAt(field, { [field]: value });
      return '';
    } catch (error) {
      return error.message;
    }
  }, [validationSchema]);

  const handleChange = useCallback((field) => (value) => {
    setValues(prev => ({ ...prev, [field]: value }));
    
    // Clear error when user starts typing
    if (errors[field]) {
      setErrors(prev => ({ ...prev, [field]: '' }));
    }
  }, [errors]);

  const handleBlur = useCallback((field) => async () => {
    setTouched(prev => ({ ...prev, [field]: true }));
    
    const error = await validateField(field, values[field]);
    setErrors(prev => ({ ...prev, [field]: error }));
  }, [values, validateField]);

  const handleSubmit = useCallback(async (e) => {
    e.preventDefault();
    setIsSubmitting(true);

    try {
      // Validate all fields
      if (validationSchema) {
        await validationSchema.validate(values, { abortEarly: false });
      }

      // Mark all fields as touched
      const touchedFields = Object.keys(values).reduce((acc, key) => {
        acc[key] = true;
        return acc;
      }, {});
      setTouched(touchedFields);

      await onSubmit(values);
    } catch (error) {
      if (error.inner) {
        // Yup validation errors
        const validationErrors = error.inner.reduce((acc, err) => {
          acc[err.path] = err.message;
          return acc;
        }, {});
        setErrors(validationErrors);
      } else {
        throw error;
      }
    } finally {
      setIsSubmitting(false);
    }
  }, [values, validationSchema, onSubmit]);

  const resetForm = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  }, [initialValues]);

  const setFieldValue = useCallback((field, value) => {
    setValues(prev => ({ ...prev, [field]: value }));
  }, []);

  const setFieldError = useCallback((field, error) => {
    setErrors(prev => ({ ...prev, [field]: error }));
  }, []);

  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    resetForm,
    setFieldValue,
    setFieldError
  };
}
```

## Error Handling Examples

### Global Error Boundary
```javascript
// src/components/ErrorBoundary.jsx
import React from 'react';
import { Alert } from './Alert';
import { Button } from './Button';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error boundary caught an error:', error, errorInfo);
    
    // Log to error reporting service
    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }
  }

  handleRetry = () => {
    this.setState({ hasError: false, error: null });
  };

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback(this.state.error, this.handleRetry);
      }

      return (
        <div className="error-boundary">
          <Alert
            variant="error"
            title="Terjadi kesalahan"
            message="Maaf, terjadi kesalahan tidak terduga. Silakan muat ulang halaman."
            actions={[
              {
                label: 'Muat Ulang',
                onClick: () => window.location.reload(),
                variant: 'primary'
              },
              {
                label: 'Coba Lagi',
                onClick: this.handleRetry,
                variant: 'secondary'
              }
            ]}
          />
        </div>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

### Error Handling Service
```javascript
// src/utils/errors.js
export class AppError extends Error {
  constructor(message, code, status) {
    super(message);
    this.name = 'AppError';
    this.code = code;
    this.status = status;
    this.timestamp = new Date();
  }
}

export function createError(message, code, status) {
  return new AppError(message, code, status);
}

export function handleApiError(error) {
  const errorMap = {
    'INVALID_CREDENTIALS': {
      message: 'Email atau password tidak valid',
      isRetryable: false,
      shouldLogout: false
    },
    'TOKEN_EXPIRED': {
      message: 'Sesi Anda telah berakhir, silakan masuk kembali',
      isRetryable: false,
      shouldLogout: true
    },
    'NETWORK_ERROR': {
      message: 'Koneksi bermasalah, silakan coba lagi',
      isRetryable: true,
      shouldLogout: false
    },
    'RATE_LIMIT_EXCEEDED': {
      message: 'Terlalu banyak percobaan, silakan tunggu sebentar',
      isRetryable: true,
      shouldLogout: false
    }
  };

  const mapped = errorMap[error.code] || {
    message: error.message || 'Terjadi kesalahan tidak terduga',
    isRetryable: false,
    shouldLogout: false
  };

  return {
    ...mapped,
    code: error.code || 'UNKNOWN_ERROR'
  };
}
```

## Testing Examples

### Component Testing
```javascript
// src/components/__tests__/LoginForm.test.js
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from '../LoginForm';

describe('LoginForm', () => {
  const mockSubmit = jest.fn();
  
  beforeEach(() => {
    mockSubmit.mockClear();
  });

  it('renders form fields correctly', () => {
    render(<LoginForm onSubmit={mockSubmit} />);
    
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /masuk/i })).toBeInTheDocument();
  });

  it('validates required fields', async () => {
    render(<LoginForm onSubmit={mockSubmit} />);
    
    const submitButton = screen.getByRole('button', { name: /masuk/i });
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByText(/email wajib diisi/i)).toBeInTheDocument();
      expect(screen.getByText(/password wajib diisi/i)).toBeInTheDocument();
    });

    expect(mockSubmit).not.toHaveBeenCalled();
  });

  it('validates email format', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={mockSubmit} />);
    
    const emailInput = screen.getByLabelText(/email/i);
    await user.type(emailInput, 'invalid-email');
    await user.tab(); // Blur the field

    await waitFor(() => {
      expect(screen.getByText(/format email tidak valid/i)).toBeInTheDocument();
    });
  });

  it('submits form with valid data', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={mockSubmit} />);
    
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /masuk/i }));

    await waitFor(() => {
      expect(mockSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123',
        rememberMe: false
      });
    });
  });

  it('shows loading state during submission', async () => {
    render(<LoginForm onSubmit={mockSubmit} isLoading={true} />);
    
    expect(screen.getByText(/masuk\.\.\./i)).toBeInTheDocument();
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

### Hook Testing
```javascript
// src/hooks/__tests__/useAuth.test.js
import { renderHook, act } from '@testing-library/react';
import { useAuth, AuthProvider } from '../useAuth';
import { authService } from '../../services/authService';

jest.mock('../../services/authService');

describe('useAuth', () => {
  const wrapper = ({ children }) => <AuthProvider>{children}</AuthProvider>;

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('starts with unauthenticated state', () => {
    const { result } = renderHook(() => useAuth(), { wrapper });
    
    expect(result.current.isAuthenticated).toBe(false);
    expect(result.current.user).toBe(null);
  });

  it('logs in user successfully', async () => {
    const mockUser = { id: '1', email: 'test@example.com' };
    authService.login.mockResolvedValue({
      token: 'fake-token',
      user: mockUser,
      expiresIn: 3600
    });

    const { result } = renderHook(() => useAuth(), { wrapper });

    await act(async () => {
      await result.current.login({
        email: 'test@example.com',
        password: 'password'
      });
    });

    expect(result.current.isAuthenticated).toBe(true);
    expect(result.current.user).toEqual(mockUser);
  });

  it('handles login failure', async () => {
    authService.login.mockRejectedValue(new Error('Invalid credentials'));

    const { result } = renderHook(() => useAuth(), { wrapper });

    await expect(
      act(async () => {
        await result.current.login({
          email: 'wrong@example.com',
          password: 'wrongpassword'
        });
      })
    ).rejects.toThrow('Invalid credentials');

    expect(result.current.isAuthenticated).toBe(false);
  });
});
```

## Styling Examples

### CSS Modules
```css
/* src/components/LoginForm.module.css */
.loginForm {
  max-width: 400px;
  margin: 0 auto;
  padding: 2rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.field {
  margin-bottom: 1.5rem;
}

.submitButton {
  width: 100%;
  margin-top: 1rem;
}

.errorMessage {
  color: #dc3545;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}

@media (max-width: 768px) {
  .loginForm {
    margin: 1rem;
    padding: 1.5rem;
  }
}
```

### Styled Components
```javascript
// src/components/styled/LoginForm.js
import styled from 'styled-components';

export const FormContainer = styled.div`
  max-width: 400px;
  margin: 0 auto;
  padding: 2rem;
  background: ${props => props.theme.colors.background};
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);

  @media (max-width: 768px) {
    margin: 1rem;
    padding: 1.5rem;
  }
`;

export const Title = styled.h1`
  text-align: center;
  margin-bottom: 2rem;
  color: ${props => props.theme.colors.text};
  font-size: 1.5rem;
  font-weight: 600;
`;

export const Field = styled.div`
  margin-bottom: 1.5rem;
`;

export const SubmitButton = styled.button`
  width: 100%;
  padding: 0.75rem;
  margin-top: 1rem;
  background: ${props => props.theme.colors.primary};
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
  
  &:hover:not(:disabled) {
    background: ${props => props.theme.colors.primaryDark};
  }
  
  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
`;
```

This comprehensive documentation provides real-world examples that developers can use as templates for implementing the login application functionality.