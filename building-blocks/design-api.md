# API Design: Building Interfaces That Don't Suck

*Last Updated: December 2024*

---

## 🧒 **Explain Like I'm 5**

**Imagine you're at a restaurant!**

**Bad Restaurant (Bad API):**
- Menu has 1000 items, no pictures 😵
- Waiter speaks a language you don't understand
- Sometimes they bring the wrong food
- No way to know if your order is ready
- Takes forever to get anything

**Good Restaurant (Good API):**
- Simple menu with clear descriptions 📋
- Waiter speaks your language clearly
- Always get exactly what you ordered ✅
- You know when food is ready
- Fast and friendly service! 😊

**An API is like that restaurant menu and waiter** - it's how programs "order" data or actions from other programs. A good API makes it easy and pleasant to get what you need!

---

## 📋 **What is an API?**

**API (Application Programming Interface)** is a contract that defines how software components communicate with each other. It specifies what requests can be made, how to make them, and what responses to expect.

### **Simple Analogy:**

```
Your App: "Hey Weather API, what's the temperature in Toronto?"
Weather API: "It's 15°C and sunny!"

Your App: "Hey Payment API, charge this card $50"
Payment API: "Done! Here's the transaction ID: abc123"
```

### **Why APIs Matter:**

```
Without APIs:
❌ Can't use external services
❌ Everything built from scratch
❌ No integration with other systems
❌ Can't share data between applications

With APIs:
✅ Use best-in-class services (Stripe, AWS, Google Maps)
✅ Build on existing platforms
✅ Mobile apps talk to backend
✅ Third parties can build on your platform
```

---

## 🏗️ **REST API Fundamentals**

**REST (Representational State Transfer)** is the most popular API architecture style.

### **Core Principles:**

```
1. Resources - Everything is a "resource" (user, post, order)
2. HTTP Methods - Actions on resources (GET, POST, PUT, DELETE)
3. Stateless - Each request is independent
4. JSON - Standard data format
5. URIs - Unique addresses for resources
```

### **HTTP Methods (CRUD Operations):**

```
┌────────────┬──────────┬─────────────────────┐
│ HTTP Method│ CRUD     │ Description         │
├────────────┼──────────┼─────────────────────┤
│ GET        │ Read     │ Retrieve data       │
│ POST       │ Create   │ Create new resource │
│ PUT        │ Update   │ Replace resource    │
│ PATCH      │ Update   │ Partial update      │
│ DELETE     │ Delete   │ Remove resource     │
└────────────┴──────────┴─────────────────────┘
```

---

## 🎨 **RESTful URL Design**

### **Good vs Bad Examples:**

```
❌ BAD:
GET  /getUser?id=123
POST /createUser
POST /updateUser
POST /deleteUser
GET  /user/delete/123

✅ GOOD:
GET    /users/123          (Get user)
POST   /users              (Create user)
PUT    /users/123          (Update user)
DELETE /users/123          (Delete user)
```

### **Resource Naming Rules:**

```
✅ Use nouns, not verbs
   /users not /getUsers

✅ Use plural for collections
   /users not /user

✅ Hierarchical relationships
   /users/123/posts
   /users/123/posts/456/comments

✅ Lowercase with hyphens
   /order-items not /OrderItems

❌ Avoid verbs in URLs
   /createUser ❌
   /users ✅
```

### **Real-World Examples:**

```javascript
// User management
GET    /users              // List all users
GET    /users/123          // Get specific user
POST   /users              // Create new user
PUT    /users/123          // Update user
DELETE /users/123          // Delete user

// Nested resources
GET    /users/123/posts              // User's posts
GET    /users/123/posts/456          // Specific post
POST   /users/123/posts              // Create post for user
GET    /users/123/followers          // User's followers

// Filtering and pagination
GET    /users?role=admin              // Filter users
GET    /users?page=2&limit=20         // Pagination
GET    /posts?sort=created_at:desc    // Sorting
GET    /products?min_price=10&max_price=100  // Range

// Search
GET    /users?search=alice            // Search users
GET    /products?q=laptop              // Text search
```

---

## 📊 **HTTP Status Codes**

**Kid-friendly:** "Status codes are like restaurant feedback - did your order work?"

### **Success (2xx):**

```
200 OK                  - Request succeeded
201 Created             - New resource created
204 No Content          - Success, but nothing to return

Example:
GET /users/123
→ 200 OK + user data

POST /users
→ 201 Created + new user data
```

### **Client Errors (4xx):**

```
400 Bad Request         - Invalid data sent
401 Unauthorized        - Not logged in
403 Forbidden           - Logged in, but not allowed
404 Not Found           - Resource doesn't exist
409 Conflict            - Duplicate or conflict
422 Unprocessable       - Validation failed
429 Too Many Requests   - Rate limit exceeded

Example:
POST /users (invalid email)
→ 400 Bad Request
{
  "error": "Invalid email format",
  "field": "email"
}
```

### **Server Errors (5xx):**

```
500 Internal Server Error  - Something broke
502 Bad Gateway            - Upstream service failed
503 Service Unavailable    - Server overloaded
504 Gateway Timeout        - Upstream timeout

Example:
GET /users/123
→ 500 Internal Server Error
{
  "error": "Database connection failed"
}
```

---

## 💻 **Complete API Example**

### **User Management API:**

```javascript
const express = require('express');
const app = express();
app.use(express.json());

// ============================================
// GET - List users with filtering & pagination
// ============================================
app.get('/api/users', async (req, res) => {
  try {
    const { 
      page = 1, 
      limit = 20, 
      role, 
      search,
      sort = 'created_at:desc' 
    } = req.query;
    
    // Build query
    let query = db.select('*').from('users');
    
    // Filtering
    if (role) {
      query = query.where('role', role);
    }
    
    if (search) {
      query = query.where('username', 'like', `%${search}%`);
    }
    
    // Sorting
    const [sortField, sortOrder] = sort.split(':');
    query = query.orderBy(sortField, sortOrder || 'asc');
    
    // Pagination
    const offset = (page - 1) * limit;
    query = query.limit(limit).offset(offset);
    
    // Execute
    const users = await query;
    const total = await db('users').count('* as count');
    
    res.json({
      data: users,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total: total[0].count,
        totalPages: Math.ceil(total[0].count / limit)
      }
    });
    
  } catch (error) {
    console.error('Error fetching users:', error);
    res.status(500).json({
      error: 'Failed to fetch users',
      message: error.message
    });
  }
});

// ============================================
// GET - Get single user by ID
// ============================================
app.get('/api/users/:id', async (req, res) => {
  try {
    const { id } = req.params;
    
    const user = await db('users')
      .where('id', id)
      .first();
    
    if (!user) {
      return res.status(404).json({
        error: 'User not found',
        userId: id
      });
    }
    
    // Don't send password!
    delete user.password;
    
    res.json({ data: user });
    
  } catch (error) {
    console.error('Error fetching user:', error);
    res.status(500).json({
      error: 'Failed to fetch user',
      message: error.message
    });
  }
});

// ============================================
// POST - Create new user
// ============================================
app.post('/api/users', async (req, res) => {
  try {
    const { username, email, password, role = 'user' } = req.body;
    
    // Validation
    if (!username || !email || !password) {
      return res.status(400).json({
        error: 'Missing required fields',
        required: ['username', 'email', 'password']
      });
    }
    
    // Email validation
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      return res.status(400).json({
        error: 'Invalid email format',
        field: 'email'
      });
    }
    
    // Check if user exists
    const existing = await db('users')
      .where('email', email)
      .orWhere('username', username)
      .first();
    
    if (existing) {
      return res.status(409).json({
        error: 'User already exists',
        field: existing.email === email ? 'email' : 'username'
      });
    }
    
    // Hash password
    const hashedPassword = await bcrypt.hash(password, 10);
    
    // Create user
    const [userId] = await db('users').insert({
      username,
      email,
      password: hashedPassword,
      role,
      created_at: new Date()
    });
    
    // Fetch created user
    const newUser = await db('users')
      .where('id', userId)
      .first();
    
    delete newUser.password;
    
    res.status(201).json({
      message: 'User created successfully',
      data: newUser
    });
    
  } catch (error) {
    console.error('Error creating user:', error);
    res.status(500).json({
      error: 'Failed to create user',
      message: error.message
    });
  }
});

// ============================================
// PUT - Update user (replace)
// ============================================
app.put('/api/users/:id', async (req, res) => {
  try {
    const { id } = req.params;
    const { username, email, role } = req.body;
    
    // Check if user exists
    const user = await db('users').where('id', id).first();
    
    if (!user) {
      return res.status(404).json({
        error: 'User not found',
        userId: id
      });
    }
    
    // Update
    await db('users')
      .where('id', id)
      .update({
        username,
        email,
        role,
        updated_at: new Date()
      });
    
    // Fetch updated user
    const updated = await db('users').where('id', id).first();
    delete updated.password;
    
    res.json({
      message: 'User updated successfully',
      data: updated
    });
    
  } catch (error) {
    console.error('Error updating user:', error);
    res.status(500).json({
      error: 'Failed to update user',
      message: error.message
    });
  }
});

// ============================================
// PATCH - Partial update user
// ============================================
app.patch('/api/users/:id', async (req, res) => {
  try {
    const { id } = req.params;
    const updates = req.body;
    
    // Don't allow updating sensitive fields
    delete updates.password;
    delete updates.id;
    delete updates.created_at;
    
    if (Object.keys(updates).length === 0) {
      return res.status(400).json({
        error: 'No valid fields to update'
      });
    }
    
    const user = await db('users').where('id', id).first();
    
    if (!user) {
      return res.status(404).json({
        error: 'User not found',
        userId: id
      });
    }
    
    // Update only provided fields
    await db('users')
      .where('id', id)
      .update({
        ...updates,
        updated_at: new Date()
      });
    
    const updated = await db('users').where('id', id).first();
    delete updated.password;
    
    res.json({
      message: 'User updated successfully',
      data: updated
    });
    
  } catch (error) {
    console.error('Error updating user:', error);
    res.status(500).json({
      error: 'Failed to update user',
      message: error.message
    });
  }
});

// ============================================
// DELETE - Delete user
// ============================================
app.delete('/api/users/:id', async (req, res) => {
  try {
    const { id } = req.params;
    
    const user = await db('users').where('id', id).first();
    
    if (!user) {
      return res.status(404).json({
        error: 'User not found',
        userId: id
      });
    }
    
    await db('users').where('id', id).delete();
    
    res.status(204).send();  // No content
    
  } catch (error) {
    console.error('Error deleting user:', error);
    res.status(500).json({
      error: 'Failed to delete user',
      message: error.message
    });
  }
});

app.listen(3000, () => {
  console.log('API server running on port 3000');
});
```

---

## 🔐 **Authentication & Authorization**

### **Authentication Methods:**

```
1. API Keys (Simple)
2. JWT Tokens (Most common)
3. OAuth 2.0 (Third-party auth)
4. Session-based (Traditional)
```

### **JWT Authentication Example:**

```javascript
const jwt = require('jsonwebtoken');

// Login endpoint
app.post('/api/auth/login', async (req, res) => {
  const { email, password } = req.body;
  
  // Verify credentials
  const user = await db('users').where('email', email).first();
  
  if (!user || !await bcrypt.compare(password, user.password)) {
    return res.status(401).json({
      error: 'Invalid credentials'
    });
  }
  
  // Generate JWT
  const token = jwt.sign(
    { 
      userId: user.id, 
      email: user.email,
      role: user.role 
    },
    process.env.JWT_SECRET,
    { expiresIn: '24h' }
  );
  
  res.json({
    message: 'Login successful',
    token,
    user: {
      id: user.id,
      username: user.username,
      email: user.email,
      role: user.role
    }
  });
});

// Authentication middleware
function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];  // Bearer TOKEN
  
  if (!token) {
    return res.status(401).json({
      error: 'Access token required'
    });
  }
  
  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({
        error: 'Invalid or expired token'
      });
    }
    
    req.user = user;  // Attach user to request
    next();
  });
}

// Protected route
app.get('/api/profile', authenticateToken, async (req, res) => {
  const user = await db('users')
    .where('id', req.user.userId)
    .first();
  
  delete user.password;
  res.json({ data: user });
});

// Authorization middleware
function requireRole(role) {
  return (req, res, next) => {
    if (req.user.role !== role) {
      return res.status(403).json({
        error: 'Insufficient permissions'
      });
    }
    next();
  };
}

// Admin-only route
app.delete('/api/users/:id', 
  authenticateToken, 
  requireRole('admin'), 
  async (req, res) => {
    // Delete user logic
  }
);
```

---

## 📈 **Rate Limiting**

**Kid-friendly:** "Like a bouncer at a club - only let so many people in per minute"

```javascript
const rateLimit = require('express-rate-limit');

// General rate limit
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // Max 100 requests per window
  message: {
    error: 'Too many requests, please try again later',
    retryAfter: '15 minutes'
  },
  standardHeaders: true,      // Return rate limit info in headers
  legacyHeaders: false
});

app.use('/api/', limiter);

// Stricter limit for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,                     // Only 5 login attempts
  skipSuccessfulRequests: true
});

app.use('/api/auth/login', authLimiter);
```

---

## 📝 **API Versioning**

**Why version?** Breaking changes shouldn't break existing clients!

### **Methods:**

```
1. URL Versioning (Most common)
   /api/v1/users
   /api/v2/users

2. Header Versioning
   Accept: application/vnd.myapi.v1+json

3. Query Parameter
   /api/users?version=1
```

### **Example:**

```javascript
// Version 1 (old)
app.get('/api/v1/users/:id', async (req, res) => {
  const user = await db('users').where('id', req.params.id).first();
  
  res.json({
    id: user.id,
    name: user.username,  // v1 uses "name"
    email: user.email
  });
});

// Version 2 (new)
app.get('/api/v2/users/:id', async (req, res) => {
  const user = await db('users').where('id', req.params.id).first();
  
  res.json({
    id: user.id,
    username: user.username,  // v2 uses "username"
    email: user.email,
    profile: {                 // v2 adds nested profile
      avatar: user.avatar,
      bio: user.bio
    }
  });
});
```

---

## 🔍 **API Documentation**

**Good API = Great documentation!**

### **OpenAPI/Swagger Example:**

```yaml
openapi: 3.0.0
info:
  title: User Management API
  version: 1.0.0
  description: API for managing users

paths:
  /users:
    get:
      summary: List all users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        200:
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
    
    post:
      summary: Create new user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - username
                - email
                - password
              properties:
                username:
                  type: string
                email:
                  type: string
                  format: email
                password:
                  type: string
                  format: password
      responses:
        201:
          description: User created
        400:
          description: Invalid input

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        username:
          type: string
        email:
          type: string
        role:
          type: string
          enum: [user, admin]
        created_at:
          type: string
          format: date-time
```

---

## 🛡️ **Error Handling Best Practices**

### **Consistent Error Format:**

```javascript
// Standard error response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address"
      },
      {
        "field": "password",
        "message": "Must be at least 8 characters"
      }
    ],
    "timestamp": "2024-12-30T10:30:00Z",
    "path": "/api/users"
  }
}
```

### **Error Handling Middleware:**

```javascript
// Custom error class
class APIError extends Error {
  constructor(statusCode, message, code, details = []) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.details = details;
  }
}

// Global error handler
app.use((err, req, res, next) => {
  // Default to 500 if not specified
  const statusCode = err.statusCode || 500;
  
  // Log error
  console.error({
    error: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method,
    timestamp: new Date()
  });
  
  // Don't leak internal errors to client
  const response = {
    error: {
      code: err.code || 'INTERNAL_ERROR',
      message: statusCode === 500 
        ? 'An internal error occurred' 
        : err.message,
      details: err.details || [],
      timestamp: new Date().toISOString(),
      path: req.path
    }
  };
  
  // Include stack trace in development
  if (process.env.NODE_ENV === 'development') {
    response.error.stack = err.stack;
  }
  
  res.status(statusCode).json(response);
});

// Usage
app.post('/api/users', async (req, res, next) => {
  try {
    // Validation
    if (!req.body.email) {
      throw new APIError(
        400,
        'Validation failed',
        'VALIDATION_ERROR',
        [{ field: 'email', message: 'Email is required' }]
      );
    }
    
    // Process...
    
  } catch (error) {
    next(error);  // Pass to error handler
  }
});
```

---

## 🤔 **Common Interview Questions**

### **Q1: REST vs GraphQL vs gRPC?**

**Answer:**
```
REST:
✅ Simple, widely understood
✅ HTTP standard methods
✅ Great for CRUD operations
❌ Over-fetching/under-fetching data
❌ Multiple requests for related data

GraphQL:
✅ Request exactly what you need
✅ Single endpoint
✅ Strong typing
❌ More complex to implement
❌ Caching harder

gRPC:
✅ Very fast (binary protocol)
✅ Bi-directional streaming
❌ Not browser-friendly
❌ Steeper learning curve

Use REST for most web APIs (simplicity wins)
```

### **Q2: How do you handle API versioning?**

**Answer:**
```
URL versioning (recommended):
/api/v1/users
/api/v2/users

Benefits:
- Clear which version is being used
- Easy to route
- Cache-friendly

When to version:
- Breaking changes (removing fields, changing types)
- NOT for adding optional fields (backward compatible)

Best practice:
- Support 2-3 versions max
- Deprecate old versions gradually
- Give clients 6-12 months to migrate
```

### **Q3: How do you secure an API?**

**Answer:**
```
1. Authentication (who are you?)
   - JWT tokens
   - OAuth 2.0
   - API keys

2. Authorization (what can you do?)
   - Role-based access control
   - Check permissions per request

3. HTTPS (always!)
   - Encrypt all traffic
   - No plain HTTP

4. Rate limiting
   - Prevent abuse
   - 100 requests per 15 min

5. Input validation
   - Sanitize all inputs
   - Prevent SQL injection

6. CORS
   - Restrict which domains can call API
```

### **Q4: How do you design for pagination?**

**Answer:**
```
Offset-based (simple):
GET /users?page=2&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 1000,
    "totalPages": 50
  }
}

Cursor-based (for large datasets):
GET /users?cursor=abc123&limit=20

Response:
{
  "data": [...],
  "cursor": {
    "next": "def456",
    "prev": "xyz789"
  }
}

Cursor-based better for:
- Real-time feeds
- Large datasets
- When items can be added/deleted
```

---

## 🎯 **API Design Checklist**

```
✅ Use RESTful URL conventions
✅ Use proper HTTP methods and status codes
✅ Return consistent error formats
✅ Implement authentication and authorization
✅ Add rate limiting
✅ Version your API
✅ Document everything (OpenAPI/Swagger)
✅ Use HTTPS everywhere
✅ Validate all inputs
✅ Implement pagination for lists
✅ Add request/response logging
✅ Set appropriate cache headers
✅ Handle CORS properly
✅ Use consistent naming (snake_case or camelCase)
❌ Don't return sensitive data (passwords, secrets)
❌ Don't trust client input
❌ Don't expose internal errors
```

---

## 🚀 **Next Steps**

1. **Practice:** Build a simple REST API with all CRUD operations
2. **Try:** Implement JWT authentication
3. **Read Next:** [Microservices Architecture](../06-system-design-patterns/microservices.md)
4. **Build:** Create API documentation with Swagger

---

## 📚 **Further Reading**

- [REST API Design Best Practices](https://restfulapi.net/)
- [OpenAPI Specification](https://swagger.io/specification/)
- [Microsoft API Guidelines](https://github.com/Microsoft/api-guidelines)
- [Google API Design Guide](https://cloud.google.com/apis/design)

---

**Next Topic:** [Microservices](../06-system-design-patterns/microservices.md)  
**Previous:** [Message Queues](./message-queues.md)

---

*Questions or improvements? Open an issue or submit a PR!*
