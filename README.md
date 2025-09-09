# Postman Cheat Sheet

Postman is a comprehensive API development and testing platform that enables developers to design, test, document, and monitor APIs. It provides a user-friendly interface for making HTTP requests and automating API testing workflows.

---

## Table of Contents
- [Installation & Setup](#installation--setup)
- [Interface Overview](#interface-overview)
- [Making Requests](#making-requests)
- [Authentication](#authentication)
- [Collections](#collections)
- [Variables](#variables)
- [Scripts and Tests](#scripts-and-tests)
- [Environments](#environments)
- [Mock Servers](#mock-servers)
- [Monitors](#monitors)
- [Newman CLI](#newman-cli)
- [Collaboration](#collaboration)
- [Best Practices](#best-practices)

---

## Installation & Setup

### Desktop Application
```bash
# Download from https://www.postman.com/downloads/
# Available for Windows, macOS, and Linux

# Alternative: Install via package managers

# macOS with Homebrew
brew install --cask postman

# Windows with Chocolatey
choco install postman

# Linux with Snap
sudo snap install postman
```

### Web Version
- Access at [web.postman.co](https://web.postman.co)
- Requires Postman account
- Limited compared to desktop app

### API Client
```bash
# Postman can also be used programmatically
npm install postman-collection newman
```

## Interface Overview

### Main Components
| Component | Description | Shortcut |
|-----------|-------------|----------|
| **Request Builder** | Create and configure requests | `Ctrl+N` |
| **Response Viewer** | View API responses | |
| **History** | Previously sent requests | `Ctrl+H` |
| **Collections** | Organized groups of requests | `Ctrl+Shift+N` |
| **Environments** | Variable sets for different contexts | `Ctrl+Shift+E` |
| **Console** | Debug and log information | `Alt+Ctrl+C` |

### Request Tabs
- **Params**: Query parameters
- **Authorization**: Authentication settings
- **Headers**: Request headers
- **Body**: Request payload
- **Pre-request Script**: Code before request
- **Tests**: Validation after response

## Making Requests

### HTTP Methods
```javascript
// GET request
GET https://api.example.com/users

// POST request with JSON body
POST https://api.example.com/users
{
  "name": "John Doe",
  "email": "john@example.com"
}

// PUT request
PUT https://api.example.com/users/123
{
  "name": "Jane Doe",
  "email": "jane@example.com"
}

// DELETE request
DELETE https://api.example.com/users/123

// PATCH request
PATCH https://api.example.com/users/123
{
  "email": "newemail@example.com"
}
```

### Request Configuration
```javascript
// Query Parameters
https://api.example.com/users?page=1&limit=10&sort=name

// Headers
Content-Type: application/json
Accept: application/json
User-Agent: Postman/7.36.0

// Authorization Header
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Request Body Types
```javascript
// JSON
{
  "user": {
    "name": "John",
    "age": 30
  }
}

// Form Data (multipart/form-data)
name: John Doe
email: john@example.com
avatar: [file upload]

// URL Encoded (application/x-www-form-urlencoded)
name=John+Doe&email=john%40example.com

// Raw Text
Plain text content here

// Binary
[Binary file upload]
```

## Authentication

### Authentication Types
```javascript
// API Key
// Add to Header or Query Param
Key: X-API-Key
Value: your-api-key-here

// Bearer Token
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// Basic Auth
// Username: admin
// Password: password123
// Generates: Authorization: Basic YWRtaW46cGFzc3dvcmQxMjM=

// Digest Auth
// Username: user
// Password: pass
// Automatically handles challenge-response

// OAuth 1.0
// Consumer Key, Consumer Secret, Access Token, Token Secret

// OAuth 2.0
// Grant Type: Authorization Code, Client Credentials, etc.
// Access Token URL, Client ID, Client Secret

// NTLM Authentication
// Username, Password, Domain, Workstation

// AWS Signature
// Access Key, Secret Key, AWS Region, Service Name
```

### Dynamic Authentication
```javascript
// Pre-request Script for JWT token
pm.test("Get auth token", function () {
    pm.sendRequest({
        url: 'https://api.example.com/auth/login',
        method: 'POST',
        header: {
            'Content-Type': 'application/json',
        },
        body: {
            mode: 'raw',
            raw: JSON.stringify({
                username: 'admin',
                password: 'password123'
            })
        }
    }, function (err, response) {
        if (response.code === 200) {
            const token = response.json().token;
            pm.environment.set('auth_token', token);
        }
    });
});
```

## Collections

### Creating Collections
```javascript
// Collection Structure
My API Collection/
├── User Management/
│   ├── Get All Users
│   ├── Get User by ID
│   ├── Create User
│   ├── Update User
│   └── Delete User
├── Authentication/
│   ├── Login
│   ├── Refresh Token
│   └── Logout
└── Data Management/
    ├── Import Data
    └── Export Data
```

### Collection Scripts
```javascript
// Collection Pre-request Script (runs before every request)
// Set base URL
pm.environment.set('base_url', 'https://api.example.com');

// Set common headers
pm.request.headers.add({
    key: 'Content-Type',
    value: 'application/json'
});

// Collection Tests (runs after every request)
// Check response status
pm.test("Status code is success", function () {
    pm.expect(pm.response.code).to.be.oneOf([200, 201, 204]);
});

// Check response time
pm.test("Response time is less than 1000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(1000);
});
```

### Collection Variables
```javascript
// Collection-level variables (available to all requests in collection)
{
  "base_url": "https://api.example.com",
  "api_version": "v1",
  "timeout": 5000
}

// Usage in requests
{{base_url}}/{{api_version}}/users
```

## Variables

### Variable Scopes
| Scope | Precedence | Usage |
|-------|------------|--------|
| **Data** | 1 (Highest) | CSV/JSON data for iterations |
| **Local** | 2 | Current request execution |
| **Environment** | 3 | Specific environment context |
| **Collection** | 4 | Shared across collection |
| **Global** | 5 (Lowest) | Across all collections |

### Working with Variables
```javascript
// Setting Variables in Scripts
pm.environment.set('user_id', '123');
pm.globals.set('api_key', 'abc123');
pm.collectionVariables.set('base_url', 'https://api.example.com');

// Getting Variables
const userId = pm.environment.get('user_id');
const apiKey = pm.globals.get('api_key');

// Using Variables in Requests
// URL: {{base_url}}/users/{{user_id}}
// Header: Authorization: Bearer {{auth_token}}

// Dynamic Variables
{{$guid}}          // Generates GUID
{{$timestamp}}     // Current timestamp
{{$randomInt}}     // Random integer
{{$randomFirstName}} // Random first name
{{$randomEmail}}   // Random email address

// Variable Resolution Example
GET {{base_url}}/users/{{user_id}}?key={{api_key}}
```

### Environment Management
```javascript
// Development Environment
{
  "base_url": "https://dev-api.example.com",
  "db_host": "dev-db.example.com",
  "debug": true
}

// Production Environment
{
  "base_url": "https://api.example.com",
  "db_host": "prod-db.example.com",
  "debug": false
}

// Switching Environments
// Use dropdown in top-right corner
// Or use Newman CLI: --environment production.json
```

## Scripts and Tests

### Pre-request Scripts
```javascript
// Set dynamic timestamp
pm.environment.set('timestamp', Date.now());

// Generate random data
pm.environment.set('random_email', 
    'user' + Math.random().toString(36).substring(7) + '@example.com'
);

// Conditional logic
if (!pm.environment.get('auth_token')) {
    // Get authentication token first
    pm.sendRequest({
        url: pm.environment.get('base_url') + '/auth/login',
        method: 'POST',
        body: {
            mode: 'raw',
            raw: JSON.stringify({
                username: pm.environment.get('username'),
                password: pm.environment.get('password')
            })
        }
    }, function (err, response) {
        pm.environment.set('auth_token', response.json().token);
    });
}

// Modify request
pm.request.url = pm.request.url + '?timestamp=' + Date.now();
```

### Test Scripts
```javascript
// Status Code Tests
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Status code name has string OK", function () {
    pm.response.to.have.status("OK");
});

// Response Time Tests
pm.test("Response time is less than 500ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// Header Tests
pm.test("Content-Type is present", function () {
    pm.response.to.have.header("Content-Type");
});

pm.test("Content-Type is application/json", function () {
    pm.expect(pm.response.headers.get('Content-Type')).to.include('application/json');
});

// Body Tests
pm.test("Response body is JSON", function () {
    pm.response.to.be.json;
});

pm.test("Response has user data", function () {
    const responseJson = pm.response.json();
    pm.expect(responseJson).to.have.property('user');
    pm.expect(responseJson.user).to.have.property('id');
    pm.expect(responseJson.user).to.have.property('name');
});

// Advanced JSON Schema Validation
const schema = {
    type: "object",
    properties: {
        user: {
            type: "object",
            properties: {
                id: { type: "number" },
                name: { type: "string" },
                email: { type: "string", format: "email" }
            },
            required: ["id", "name", "email"]
        }
    }
};

pm.test("Schema is valid", function () {
    pm.expect(tv4.validate(pm.response.json(), schema)).to.be.true;
});

// Store Response Data
pm.test("Save user ID for next request", function () {
    const responseJson = pm.response.json();
    pm.environment.set('created_user_id', responseJson.user.id);
});

// Conditional Tests
pm.test("Error handling", function () {
    if (pm.response.code !== 200) {
        const responseJson = pm.response.json();
        pm.expect(responseJson).to.have.property('error');
        console.log('Error:', responseJson.error);
    }
});
```

### Test Utilities
```javascript
// Common Test Functions
function validateResponse(expectedStatus, requiredFields) {
    pm.test(`Status code is ${expectedStatus}`, function () {
        pm.response.to.have.status(expectedStatus);
    });
    
    if (requiredFields && pm.response.code === expectedStatus) {
        const responseJson = pm.response.json();
        requiredFields.forEach(field => {
            pm.test(`Response has ${field}`, function () {
                pm.expect(responseJson).to.have.property(field);
            });
        });
    }
}

// Usage
validateResponse(200, ['user', 'token', 'expires']);

// Custom Chai Assertions
pm.test("Custom assertion", function () {
    const responseJson = pm.response.json();
    pm.expect(responseJson.users).to.be.an('array').that.is.not.empty;
    pm.expect(responseJson.users[0]).to.have.all.keys('id', 'name', 'email');
});
```

## Environments

### Environment Templates
```json
// Base Environment Template
{
  "id": "base-template",
  "name": "Base Template",
  "values": [
    {
      "key": "base_url",
      "value": "",
      "enabled": true
    },
    {
      "key": "api_version",
      "value": "v1",
      "enabled": true
    },
    {
      "key": "timeout",
      "value": "5000",
      "enabled": true
    }
  ]
}

// Development Environment
{
  "id": "dev-env",
  "name": "Development",
  "values": [
    {
      "key": "base_url",
      "value": "https://dev-api.example.com",
      "enabled": true
    },
    {
      "key": "username",
      "value": "dev_user",
      "enabled": true
    },
    {
      "key": "password",
      "value": "dev_password",
      "enabled": true,
      "type": "secret"
    }
  ]
}
```

### Environment Scripts
```javascript
// Global Pre-request Script
// Set environment-specific configurations
const env = pm.environment.name;

if (env === 'Development') {
    pm.environment.set('debug_mode', 'true');
    pm.environment.set('log_level', 'debug');
} else if (env === 'Production') {
    pm.environment.set('debug_mode', 'false');
    pm.environment.set('log_level', 'error');
}
```

## Mock Servers

### Creating Mock Servers
```javascript
// Mock Response Examples
{
  "users": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com",
      "created_at": "2023-01-01T00:00:00Z"
    },
    {
      "id": 2,
      "name": "Jane Smith", 
      "email": "jane@example.com",
      "created_at": "2023-01-02T00:00:00Z"
    }
  ],
  "total": 2,
  "page": 1,
  "limit": 10
}

// Dynamic Mock Responses
{
  "user": {
    "id": "{{$randomInt}}",
    "name": "{{$randomFullName}}",
    "email": "{{$randomEmail}}",
    "created_at": "{{$isoTimestamp}}"
  }
}
```

### Mock Server Configuration
```javascript
// Response Headers
Content-Type: application/json
X-Powered-By: Postman Mock Server
Access-Control-Allow-Origin: *

// Status Codes and Responses
// 200 OK - Successful request
// 201 Created - Resource created
// 400 Bad Request - Client error
// 404 Not Found - Resource not found
// 500 Internal Server Error - Server error

// Conditional Responses based on request
if (request.method === 'POST') {
    return {
        status: 201,
        body: {
            message: "User created successfully",
            user: { id: "{{$randomInt}}", name: request.body.name }
        }
    };
}
```

## Monitors

### Setting up Monitors
```javascript
// Monitor Configuration
{
  "name": "API Health Check",
  "collection": "api-collection-id",
  "environment": "production-env-id",
  "schedule": {
    "cron": "0 */5 * * * *",  // Every 5 minutes
    "timezone": "America/New_York"
  },
  "notifications": {
    "onError": ["email1@example.com", "email2@example.com"],
    "onFailure": ["webhook-url"]
  }
}

// Monitor Results Processing
pm.test("API is healthy", function () {
    pm.response.to.have.status(200);
    const responseJson = pm.response.json();
    pm.expect(responseJson.status).to.eql('healthy');
});

// Custom Monitor Metrics
pm.test("Response time SLA", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});

pm.test("Database connection", function () {
    const responseJson = pm.response.json();
    pm.expect(responseJson.database.connected).to.be.true;
});
```

## Newman CLI

### Installation and Basic Usage
```bash
# Install Newman
npm install -g newman

# Run collection
newman run collection.json

# Run with environment
newman run collection.json -e environment.json

# Run with data file
newman run collection.json -d test-data.csv

# Run with specific folder
newman run collection.json --folder "User Tests"

# Run with reporters
newman run collection.json --reporters cli,json,html

# Run with custom options
newman run collection.json \
  --environment prod.json \
  --reporters html,cli \
  --reporter-html-export report.html \
  --iteration-count 5 \
  --delay-request 1000
```

### Advanced Newman Usage
```bash
# CI/CD Integration
newman run collection.json \
  --environment production.json \
  --reporters junit,json \
  --reporter-junit-export results.xml \
  --reporter-json-export results.json \
  --suppress-exit-code

# Custom Newman Script
# package.json
{
  "scripts": {
    "test:api": "newman run tests/api-collection.json -e tests/environments/staging.json",
    "test:smoke": "newman run tests/smoke-tests.json --folder 'Critical Path'",
    "test:load": "newman run tests/load-test.json -n 100"
  }
}
```

## Collaboration

### Team Features
```javascript
// Workspace Management
- Public Workspaces: Visible to everyone
- Team Workspaces: Visible to team members
- Private Workspaces: Personal workspace

// Sharing Collections
- Share via link
- Invite team members
- Set permissions (viewer, editor, admin)

// Version Control
- Fork collections
- Merge requests
- Version history
- Comments and discussions

// API Documentation
- Auto-generated from collections
- Custom documentation
- Examples and descriptions
- Public documentation publishing
```

### Documentation Best Practices
```markdown
# API Documentation Structure

## Authentication
Description of authentication methods and examples.

## Endpoints

### GET /users
Retrieve a list of users.

**Parameters:**
- `page` (integer, optional): Page number
- `limit` (integer, optional): Items per page

**Example Response:**
```json
{
  "users": [...],
  "total": 100,
  "page": 1
}
```

**Error Responses:**
- `400 Bad Request`: Invalid parameters
- `401 Unauthorized`: Authentication required
```

## Best Practices

### Request Organization
```javascript
// Folder Structure Best Practices
API Collection/
├── 🔐 Authentication/
├── 👥 User Management/
│   ├── ✅ Get All Users
│   ├── ✅ Get User by ID  
│   ├── ➕ Create User
│   ├── 🔄 Update User
│   └── ❌ Delete User
├── 📊 Data Management/
└── 🧪 Test Utilities/

// Naming Conventions
- Use descriptive names
- Include HTTP method prefix
- Use emojis for visual organization
- Group related requests in folders
```

### Variable Management
```javascript
// Variable Naming Best Practices
// ✅ Good
base_url, api_version, user_id, auth_token

// ❌ Avoid
url, version, id, token

// Security Best Practices
// Use environment variables for sensitive data
// Mark sensitive variables as "secret"
// Don't commit environments with secrets to version control

// Variable Organization
// Global: Cross-collection constants
// Environment: Environment-specific values
// Collection: Collection-specific settings
// Local: Request-specific temporary values
```

### Testing Strategy
```javascript
// Test Pyramid for APIs
// 1. Contract Tests (Schema validation)
// 2. Integration Tests (End-to-end workflows)
// 3. Unit Tests (Individual endpoints)

// Test Categories
// ✅ Happy Path Tests
pm.test("Successful user creation", function () {
    pm.response.to.have.status(201);
    const user = pm.response.json().user;
    pm.expect(user).to.have.property('id');
});

// ❌ Error Path Tests  
pm.test("Handle invalid email format", function () {
    pm.response.to.have.status(400);
    const error = pm.response.json();
    pm.expect(error.message).to.include('Invalid email');
});

// 🔒 Security Tests
pm.test("Unauthorized access blocked", function () {
    pm.response.to.have.status(401);
});

// ⚡ Performance Tests
pm.test("Response time acceptable", function () {
    pm.expect(pm.response.responseTime).to.be.below(1000);
});
```

### Script Optimization
```javascript
// Efficient Script Patterns
// ✅ Cache commonly used values
const responseJson = pm.response.json();
const userId = responseJson.user.id;

// ✅ Use helper functions
function validateUser(user) {
    pm.expect(user).to.have.property('id');
    pm.expect(user).to.have.property('name');
    pm.expect(user).to.have.property('email');
}

// ✅ Handle errors gracefully
try {
    const responseJson = pm.response.json();
    validateUser(responseJson.user);
} catch (error) {
    pm.test("Response parsing failed", function () {
        pm.expect.fail('Invalid JSON response');
    });
}

// ❌ Avoid synchronous requests in scripts
// ❌ Don't make unnecessary API calls
// ❌ Avoid complex logic in scripts
```

---

## Resources
- [Postman Learning Center](https://learning.postman.com/)
- [Postman API Documentation](https://documenter.getpostman.com/)
- [Newman Documentation](https://github.com/postmanlabs/newman)
- [Postman Community](https://community.postman.com/)
- [Postman Public API Network](https://www.postman.com/explore)

---
*Originally compiled from various sources. Contributions welcome!*