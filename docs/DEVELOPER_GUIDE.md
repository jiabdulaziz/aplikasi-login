# Developer Guide

## Overview
This guide provides comprehensive instructions for setting up, developing, testing, and deploying the Login Application (aplikasi-login).

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Project Setup](#project-setup)
3. [Development Environment](#development-environment)
4. [Project Structure](#project-structure)
5. [Development Workflow](#development-workflow)
6. [Testing](#testing)
7. [Code Quality](#code-quality)
8. [Deployment](#deployment)
9. [Contributing](#contributing)
10. [Troubleshooting](#troubleshooting)

## Prerequisites

### Required Software
- **Node.js** (v16.0.0 or higher)
- **npm** (v8.0.0 or higher) or **yarn** (v1.22.0 or higher)
- **Git** (v2.25.0 or higher)
- **PostgreSQL** (v13.0 or higher) or **MySQL** (v8.0 or higher)
- **Redis** (v6.0.0 or higher) - for session storage

### Recommended Tools
- **VS Code** with extensions:
  - ES7+ React/Redux/React-Native snippets
  - Prettier - Code formatter
  - ESLint
  - GitLens
  - Auto Rename Tag
  - Bracket Pair Colorizer
- **Postman** or **Insomnia** for API testing
- **Docker** (optional, for containerized development)

## Project Setup

### 1. Clone the Repository
```bash
git clone https://github.com/your-org/aplikasi-login.git
cd aplikasi-login
```

### 2. Install Dependencies
```bash
# Using npm
npm install

# Using yarn
yarn install
```

### 3. Environment Configuration
Create environment files for different environments:

#### Frontend (.env)
```env
# API Configuration
REACT_APP_API_URL=http://localhost:3001/api/v1
REACT_APP_ENVIRONMENT=development

# Authentication
REACT_APP_JWT_SECRET=your-jwt-secret-key
REACT_APP_TOKEN_EXPIRY=3600

# External Services
REACT_APP_GOOGLE_CLIENT_ID=your-google-oauth-client-id
REACT_APP_FACEBOOK_APP_ID=your-facebook-app-id

# Analytics (optional)
REACT_APP_GA_TRACKING_ID=your-google-analytics-id
```

#### Backend (.env)
```env
# Server Configuration
NODE_ENV=development
PORT=3001
HOST=localhost

# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=aplikasi_login
DB_USER=your-db-user
DB_PASSWORD=your-db-password

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=your-redis-password

# JWT Configuration
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRY=1h
JWT_REFRESH_EXPIRY=7d

# Email Service
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password

# Security
BCRYPT_ROUNDS=12
RATE_LIMIT_WINDOW=15
RATE_LIMIT_MAX_REQUESTS=100

# Logging
LOG_LEVEL=debug
LOG_FILE=logs/app.log
```

### 4. Database Setup

#### PostgreSQL
```sql
-- Create database
CREATE DATABASE aplikasi_login;

-- Create user
CREATE USER login_user WITH ENCRYPTED PASSWORD 'your-password';
GRANT ALL PRIVILEGES ON DATABASE aplikasi_login TO login_user;
```

#### Run Migrations
```bash
# Using npm
npm run db:migrate

# Using yarn
yarn db:migrate
```

#### Seed Initial Data
```bash
# Using npm
npm run db:seed

# Using yarn
yarn db:seed
```

## Development Environment

### Starting the Development Server

#### Frontend (React)
```bash
# Start the React development server
npm run dev
# or
yarn dev

# The app will be available at http://localhost:3000
```

#### Backend (Node.js)
```bash
# Start the API server
npm run server:dev
# or
yarn server:dev

# The API will be available at http://localhost:3001
```

#### Full Stack (Concurrent)
```bash
# Start both frontend and backend
npm run dev:full
# or
yarn dev:full
```

### Hot Reloading
- **Frontend**: React Fast Refresh is enabled by default
- **Backend**: Nodemon automatically restarts on file changes

## Project Structure

```
aplikasi-login/
├── docs/                           # Documentation
│   ├── API.md                     # API documentation
│   ├── COMPONENTS.md              # Component documentation
│   ├── FUNCTIONS.md               # Function documentation
│   ├── EXAMPLES.md                # Usage examples
│   └── DEVELOPER_GUIDE.md         # This file
├── frontend/                       # React frontend
│   ├── public/
│   ├── src/
│   │   ├── components/            # Reusable components
│   │   │   ├── ui/               # Basic UI components
│   │   │   ├── forms/            # Form components
│   │   │   └── layout/           # Layout components
│   │   ├── pages/                # Page components
│   │   ├── hooks/                # Custom hooks
│   │   ├── services/             # API services
│   │   ├── utils/                # Utility functions
│   │   ├── contexts/             # React contexts
│   │   ├── styles/               # Global styles
│   │   └── __tests__/            # Test files
│   ├── package.json
│   └── README.md
├── backend/                        # Node.js backend
│   ├── src/
│   │   ├── controllers/          # Route controllers
│   │   ├── middleware/           # Express middleware
│   │   ├── models/               # Database models
│   │   ├── routes/               # API routes
│   │   ├── services/             # Business logic
│   │   ├── utils/                # Utility functions
│   │   ├── config/               # Configuration files
│   │   └── __tests__/            # Test files
│   ├── migrations/               # Database migrations
│   ├── seeds/                    # Database seeds
│   ├── package.json
│   └── README.md
├── docker/                         # Docker configuration
├── scripts/                        # Build and deployment scripts
├── .github/                        # GitHub workflows
├── package.json                    # Root package.json
└── README.md                       # Main README
```

## Development Workflow

### 1. Feature Development
```bash
# Create feature branch
git checkout -b feature/login-validation

# Make changes and commit
git add .
git commit -m "feat: add email validation to login form"

# Push branch
git push origin feature/login-validation

# Create pull request
```

### 2. Commit Message Convention
Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Build process or auxiliary tool changes

**Examples:**
```bash
feat(auth): add password strength validation
fix(api): resolve login endpoint timeout issue
docs(readme): update installation instructions
test(login): add unit tests for login form
```

### 3. Code Review Process
1. Create pull request with descriptive title and description
2. Ensure all tests pass
3. Request review from team members
4. Address feedback and update code
5. Merge after approval

## Testing

### Frontend Testing

#### Unit Tests (Jest + React Testing Library)
```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage
```

#### Example Test
```javascript
// src/components/__tests__/LoginForm.test.js
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { LoginForm } from '../LoginForm';

test('validates required fields', async () => {
  const mockSubmit = jest.fn();
  render(<LoginForm onSubmit={mockSubmit} />);
  
  fireEvent.click(screen.getByRole('button', { name: /masuk/i }));
  
  await waitFor(() => {
    expect(screen.getByText(/email wajib diisi/i)).toBeInTheDocument();
  });
});
```

#### Integration Tests
```bash
# Run integration tests
npm run test:integration
```

#### E2E Tests (Cypress)
```bash
# Open Cypress Test Runner
npm run cypress:open

# Run headless tests
npm run cypress:run
```

### Backend Testing

#### Unit Tests (Jest)
```bash
# Run all tests
npm run test

# Run specific test file
npm run test auth.controller.test.js

# Run with coverage
npm run test:coverage
```

#### Example Test
```javascript
// src/controllers/__tests__/auth.controller.test.js
const request = require('supertest');
const app = require('../../app');

describe('POST /auth/login', () => {
  it('should authenticate user with valid credentials', async () => {
    const response = await request(app)
      .post('/auth/login')
      .send({
        email: 'test@example.com',
        password: 'password123'
      });
    
    expect(response.status).toBe(200);
    expect(response.body.success).toBe(true);
    expect(response.body.data.token).toBeDefined();
  });
});
```

#### API Tests
```bash
# Run API tests
npm run test:api
```

### Test Coverage Requirements
- **Minimum**: 80% code coverage
- **Target**: 90% code coverage
- All critical paths must be tested
- New features require accompanying tests

## Code Quality

### Linting (ESLint)
```bash
# Run linter
npm run lint

# Fix auto-fixable issues
npm run lint:fix
```

#### ESLint Configuration (.eslintrc.js)
```javascript
module.exports = {
  extends: [
    'react-app',
    'react-app/jest',
    '@typescript-eslint/recommended',
    'prettier'
  ],
  rules: {
    'no-console': 'warn',
    'prefer-const': 'error',
    'react-hooks/exhaustive-deps': 'warn'
  }
};
```

### Code Formatting (Prettier)
```bash
# Format code
npm run format

# Check formatting
npm run format:check
```

#### Prettier Configuration (.prettierrc)
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

### Type Checking (TypeScript)
```bash
# Type check
npm run type-check

# Type check in watch mode
npm run type-check:watch
```

### Pre-commit Hooks (Husky)
Automatically runs before commits:
- ESLint
- Prettier
- Type checking
- Unit tests

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "git add"
    ]
  }
}
```

## Deployment

### Environment Setup

#### Staging
```bash
# Build for staging
npm run build:staging

# Deploy to staging
npm run deploy:staging
```

#### Production
```bash
# Build for production
npm run build:prod

# Deploy to production
npm run deploy:prod
```

### Docker Deployment

#### Build Docker Images
```bash
# Build frontend image
docker build -f docker/frontend/Dockerfile -t aplikasi-login-frontend .

# Build backend image
docker build -f docker/backend/Dockerfile -t aplikasi-login-backend .
```

#### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: docker/frontend/Dockerfile
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://backend:3001/api/v1

  backend:
    build:
      context: .
      dockerfile: docker/backend/Dockerfile
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:13
    environment:
      - POSTGRES_DB=aplikasi_login
      - POSTGRES_USER=login_user
      - POSTGRES_PASSWORD=secure_password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:6-alpine
    command: redis-server --requirepass secure_password

volumes:
  postgres_data:
```

### CI/CD Pipeline (GitHub Actions)

#### .github/workflows/ci.yml
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '16'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run tests
      run: npm run test:coverage
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v2

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy to production
      run: |
        # Add deployment commands here
        echo "Deploying to production..."
```

## Contributing

### Getting Started
1. Fork the repository
2. Clone your fork
3. Create a feature branch
4. Make your changes
5. Add tests for new functionality
6. Ensure all tests pass
7. Submit a pull request

### Code Standards
- Follow existing code style
- Write meaningful commit messages
- Add JSDoc comments for functions
- Update documentation for new features
- Maintain test coverage above 80%

### Pull Request Guidelines
- Use descriptive title and description
- Reference related issues
- Include screenshots for UI changes
- Ensure CI passes
- Request review from maintainers

### Issue Reporting
When reporting issues, include:
- Steps to reproduce
- Expected behavior
- Actual behavior
- Environment details
- Error messages/logs

## Troubleshooting

### Common Issues

#### 1. Port Already in Use
```bash
# Kill process using port 3000
npx kill-port 3000

# Or use different port
PORT=3001 npm start
```

#### 2. Database Connection Failed
```bash
# Check PostgreSQL status
sudo service postgresql status

# Restart PostgreSQL
sudo service postgresql restart

# Check connection
psql -h localhost -U login_user -d aplikasi_login
```

#### 3. Module Not Found
```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

#### 4. CORS Issues
```javascript
// backend/src/app.js
app.use(cors({
  origin: process.env.FRONTEND_URL || 'http://localhost:3000',
  credentials: true
}));
```

#### 5. JWT Token Issues
```bash
# Check token expiry
# Ensure JWT_SECRET is consistent across environments
# Verify token format in browser storage
```

### Performance Issues

#### 1. Slow Build Times
```bash
# Enable build cache
export CI=false

# Use faster source maps in development
GENERATE_SOURCEMAP=false npm start
```

#### 2. Large Bundle Size
```bash
# Analyze bundle
npm run build
npx serve -s build

# Use webpack-bundle-analyzer
npm install --save-dev webpack-bundle-analyzer
npm run analyze
```

### Debugging

#### Frontend Debugging
```javascript
// Enable React DevTools
// Use browser debugger
debugger;

// Console logging
console.log('Debug info:', data);

// React error boundaries
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

#### Backend Debugging
```javascript
// Use Node.js debugger
node --inspect-brk src/server.js

// Debug with VS Code
// Add breakpoints and use F5

// Logging
const logger = require('./utils/logger');
logger.debug('Debug message', { data });
```

### Getting Help
- Check existing issues on GitHub
- Review documentation
- Ask questions in discussions
- Contact maintainers via email

## Additional Resources
- [React Documentation](https://reactjs.org/docs)
- [Node.js Documentation](https://nodejs.org/docs)
- [Express.js Guide](https://expressjs.com/guide)
- [Testing Library](https://testing-library.com/docs)
- [Jest Documentation](https://jestjs.io/docs)
- [PostgreSQL Documentation](https://www.postgresql.org/docs)