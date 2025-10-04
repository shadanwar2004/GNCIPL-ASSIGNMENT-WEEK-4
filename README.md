# Leave Management System - Backend API

A complete RESTful API for employee leave management built with Node.js, Express.js, and MongoDB. This system provides user authentication with role-based access control and comprehensive leave management functionality for employees, managers, and administrators.

## 🚀 Features

- **User Authentication**: JWT-based authentication system with role-based access control
- **Leave Management**: Complete CRUD operations for leave requests
- **Role-Based Access**: Employee, Manager, and Admin roles with different permissions
- **Leave Types**: Support for sick leave, casual leave, and earned leave
- **Leave Status Tracking**: Pending, approved, and rejected status management
- **Leave Balance**: Track employee leave balance (default: 20 days per year)

## 📋 Table of Contents

- [Installation & Setup](#installation--setup)
- [Environment Variables](#environment-variables)
- [API Endpoints](#api-endpoints)
  - [Authentication Routes](#authentication-routes)
  - [Leave Management Routes](#leave-management-routes)
- [Data Models](#data-models)
- [Authentication & Authorization](#authentication--authorization)
- [Role-Based Permissions](#role-based-permissions)
- [Postman Testing](#postman-testing)
- [Error Handling](#error-handling)
- [Troubleshooting](#troubleshooting)

## 🛠️ Installation & Setup

### Prerequisites
- Node.js (v14 or higher)
- MongoDB (local or MongoDB Atlas)
- npm or yarn package manager

### Installation Steps

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd assignment4
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Create environment file**
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

4. **Start the server**
   ```bash
   # Development mode
   npm run dev

   # Production mode
   npm start
   ```

The server will run on `http://localhost:5000`

## 🔧 Environment Variables

Create a `.env` file in the root directory:

```bash
# Server Configuration
PORT=5000

# JWT Configuration
JWT_SECRET="your_super_secure_jwt_secret_key"

# MongoDB Configuration
MONGODB_URI="mongodb+srv://username:password@cluster.mongodb.net/leavemanagement"
# Or for local MongoDB:
# MONGODB_URI="mongodb://localhost:27017/leavemanagement"

# Optional: Environment
NODE_ENV="development"
```

## 📡 API Endpoints

### Base URL
```
http://localhost:5000/api
```

---

## 🔐 Authentication Routes

### 1. User Registration
**POST** `/api/auth/register`

Create a new user account with specified role.

#### Request Body
```json
{
  "name": "John Doe",
  "email": "john.doe@company.com",
  "password": "password123",
  "role": "employee"
}
```

#### Role Options
- `employee` (default) - Can apply for leaves, view own leaves
- `manager` - Can approve/reject leaves + employee permissions
- `admin` - Full system access + manager permissions

#### Success Response (201 Created)
```json
{
  "_id": "64f5a1b2c3d4e5f6789abcde",
  "name": "John Doe",
  "email": "john.doe@company.com",
  "role": "employee",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### Error Responses
- `400` - User already exists
- `500` - Internal server error

---

### 2. User Login
**POST** `/api/auth/login`

Authenticate user and get JWT token.

#### Request Body
```json
{
  "email": "john.doe@company.com",
  "password": "password123"
}
```

#### Success Response (200 OK)
```json
{
  "_id": "64f5a1b2c3d4e5f6789abcde",
  "name": "John Doe",
  "email": "john.doe@company.com",
  "role": "employee",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### Error Responses
- `400` - Invalid credentials
- `500` - Internal server error

---

## 📝 Leave Management Routes

### 1. Apply for Leave
**POST** `/api/leaves/apply`

Submit a new leave request (Employee access required).

#### Headers Required
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

#### Request Body
```json
{
  "leaveType": "casual",
  "startDate": "2024-02-15T00:00:00.000Z",
  "endDate": "2024-02-17T00:00:00.000Z",
  "reason": "Family vacation"
}
```

#### Leave Types
- `sick` - Sick leave
- `casual` - Casual leave
- `earned` - Earned leave

#### Success Response (201 Created)
```json
{
  "_id": "64f5a1b2c3d4e5f6789abce0",
  "employee": "64f5a1b2c3d4e5f6789abcde",
  "leaveType": "casual",
  "startDate": "2024-02-15T00:00:00.000Z",
  "endDate": "2024-02-17T00:00:00.000Z",
  "reason": "Family vacation",
  "status": "pending",
  "createdAt": "2024-01-15T10:30:00.000Z",
  "updatedAt": "2024-01-15T10:30:00.000Z"
}
```

#### Error Responses
- `401` - Unauthorized
- `500` - Internal server error

---

### 2. Get My Leave Requests
**GET** `/api/leaves/my`

Retrieve all leave requests for the authenticated user.

#### Headers Required
```
Authorization: Bearer <jwt_token>
```

#### Success Response (200 OK)
```json
[
  {
    "_id": "64f5a1b2c3d4e5f6789abce0",
    "employee": "64f5a1b2c3d4e5f6789abcde",
    "leaveType": "casual",
    "startDate": "2024-02-15T00:00:00.000Z",
    "endDate": "2024-02-17T00:00:00.000Z",
    "reason": "Family vacation",
    "status": "pending",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  },
  {
    "_id": "64f5a1b2c3d4e5f6789abce1",
    "employee": "64f5a1b2c3d4e5f6789abcde",
    "leaveType": "sick",
    "startDate": "2024-01-20T00:00:00.000Z",
    "endDate": "2024-01-22T00:00:00.000Z",
    "reason": "Medical appointment",
    "status": "approved",
    "approvedBy": "64f5a1b2c3d4e5f6789abcdf",
    "createdAt": "2024-01-10T09:15:00.000Z",
    "updatedAt": "2024-01-12T14:30:00.000Z"
  }
]
```

#### Error Responses
- `401` - Unauthorized
- `500` - Internal server error

---

### 3. Cancel Leave Request
**DELETE** `/api/leaves/:id`

Cancel a pending leave request (only pending leaves can be cancelled).

#### URL Parameters
- `id`: MongoDB ObjectId of the leave request

#### Headers Required
```
Authorization: Bearer <jwt_token>
```

#### Success Response (200 OK)
```json
{
  "message": "Leave cancelled"
}
```

#### Error Responses
- `400` - Only pending leaves can be cancelled
- `403` - Not authorized (not your leave request)
- `404` - Leave not found
- `401` - Unauthorized
- `500` - Internal server error

---

## 👨‍💼 Manager/Admin Routes

### 4. Get All Leave Requests
**GET** `/api/leaves/all`

Retrieve all leave requests in the system (Manager/Admin access required).

#### Headers Required
```
Authorization: Bearer <jwt_token>
```

#### Success Response (200 OK)
```json
[
  {
    "_id": "64f5a1b2c3d4e5f6789abce0",
    "employee": {
      "_id": "64f5a1b2c3d4e5f6789abcde",
      "name": "John Doe",
      "email": "john.doe@company.com",
      "role": "employee"
    },
    "leaveType": "casual",
    "startDate": "2024-02-15T00:00:00.000Z",
    "endDate": "2024-02-17T00:00:00.000Z",
    "reason": "Family vacation",
    "status": "pending",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  }
]
```

#### Error Responses
- `401` - Unauthorized
- `403` - Forbidden (insufficient permissions)
- `500` - Internal server error

---

### 5. Approve Leave Request
**PUT** `/api/leaves/:id/approve`

Approve a pending leave request (Manager/Admin access required).

#### URL Parameters
- `id`: MongoDB ObjectId of the leave request

#### Headers Required
```
Authorization: Bearer <jwt_token>
```

#### Success Response (200 OK)
```json
{
  "message": "Leave approved",
  "leave": {
    "_id": "64f5a1b2c3d4e5f6789abce0",
    "employee": "64f5a1b2c3d4e5f6789abcde",
    "leaveType": "casual",
    "startDate": "2024-02-15T00:00:00.000Z",
    "endDate": "2024-02-17T00:00:00.000Z",
    "reason": "Family vacation",
    "status": "approved",
    "approvedBy": "64f5a1b2c3d4e5f6789abcdf",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-17T11:45:00.000Z"
  }
}
```

#### Error Responses
- `401` - Unauthorized
- `403` - Forbidden (insufficient permissions)
- `404` - Leave not found
- `500` - Internal server error

---

### 6. Reject Leave Request
**PUT** `/api/leaves/:id/reject`

Reject a pending leave request (Manager/Admin access required).

#### URL Parameters
- `id`: MongoDB ObjectId of the leave request

#### Headers Required
```
Authorization: Bearer <jwt_token>
```

#### Success Response (200 OK)
```json
{
  "message": "Leave rejected",
  "leave": {
    "_id": "64f5a1b2c3d4e5f6789abce0",
    "employee": "64f5a1b2c3d4e5f6789abcde",
    "leaveType": "casual",
    "startDate": "2024-02-15T00:00:00.000Z",
    "endDate": "2024-02-17T00:00:00.000Z",
    "reason": "Family vacation",
    "status": "rejected",
    "approvedBy": "64f5a1b2c3d4e5f6789abcdf",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-17T11:45:00.000Z"
  }
}
```

#### Error Responses
- `401` - Unauthorized
- `403` - Forbidden (insufficient permissions)
- `404` - Leave not found
- `500` - Internal server error

---

## 📊 Data Models

### User Schema
```javascript
{
  _id: ObjectId,
  name: String (required),
  email: String (required, unique),
  password: String (required, hashed with bcrypt),
  role: String (enum: ["employee", "manager", "admin"], default: "employee"),
  leaveBalance: Number (default: 20),
  createdAt: Date (auto-generated),
  updatedAt: Date (auto-generated)
}
```

### Leave Request Schema
```javascript
{
  _id: ObjectId,
  employee: ObjectId (ref: "User", required),
  leaveType: String (enum: ["sick", "casual", "earned"], required),
  startDate: Date (required),
  endDate: Date (required),
  reason: String (optional),
  status: String (enum: ["pending", "approved", "rejected"], default: "pending"),
  approvedBy: ObjectId (ref: "User", optional),
  createdAt: Date (auto-generated),
  updatedAt: Date (auto-generated)
}
```

---

## 🔐 Authentication & Authorization

This API uses JWT (JSON Web Tokens) for authentication and role-based authorization:

1. **Registration/Login**: User receives a JWT token upon successful authentication
2. **Protected Routes**: Include the token in the Authorization header
3. **Token Format**: `Authorization: Bearer <your_jwt_token>`
4. **Role-Based Access**: Different endpoints require different role levels

### Security Features
- Password hashing using bcryptjs (salt rounds: 10)
- JWT token-based authentication
- Role-based access control (RBAC)
- User-specific data access for employees
- Manager/Admin level permissions for leave management

---

## 👥 Role-Based Permissions

### Employee Role
- ✅ Apply for leave requests
- ✅ View own leave requests
- ✅ Cancel own pending leave requests
- ❌ View other employees' leaves
- ❌ Approve/reject leave requests

### Manager Role
- ✅ All employee permissions
- ✅ View all leave requests
- ✅ Approve leave requests
- ✅ Reject leave requests

### Admin Role
- ✅ All manager permissions
- ✅ Full system access
- ✅ User management capabilities

---

## 🧪 Postman Testing

### Setting Up Postman Collection

1. **Create a new Postman Collection** named "Leave Management API"

2. **Set up Environment Variables**:
   - `baseUrl`: `http://localhost:5000/api`
   - `authToken`: (will be set automatically after login)
   - `leaveId`: (will be set automatically after creating leave)

### Testing Workflow

#### Step 1: Register Employee
```
POST {{baseUrl}}/auth/register
Content-Type: application/json

{
  "name": "John Employee",
  "email": "employee@company.com",
  "password": "employee123",
  "role": "employee"
}
```

#### Step 2: Register Manager
```
POST {{baseUrl}}/auth/register
Content-Type: application/json

{
  "name": "Jane Manager",
  "email": "manager@company.com",
  "password": "manager123",
  "role": "manager"
}
```

#### Step 3: Login as Employee
```
POST {{baseUrl}}/auth/login
Content-Type: application/json

{
  "email": "employee@company.com",
  "password": "employee123"
}
```

**Postman Script (Tests tab)**:
```javascript
if (pm.response.code === 200) {
    const response = pm.response.json();
    pm.environment.set("authToken", response.token);
    pm.environment.set("employeeId", response._id);
}
```

#### Step 4: Apply for Leave (Employee)
```
POST {{baseUrl}}/leaves/apply
Authorization: Bearer {{authToken}}
Content-Type: application/json

{
  "leaveType": "casual",
  "startDate": "2024-03-15T00:00:00.000Z",
  "endDate": "2024-03-17T00:00:00.000Z",
  "reason": "Personal work"
}
```

**Postman Script (Tests tab)**:
```javascript
if (pm.response.code === 201) {
    const response = pm.response.json();
    pm.environment.set("leaveId", response._id);
}
```

#### Step 5: Get My Leaves (Employee)
```
GET {{baseUrl}}/leaves/my
Authorization: Bearer {{authToken}}
```

#### Step 6: Login as Manager
```
POST {{baseUrl}}/auth/login
Content-Type: application/json

{
  "email": "manager@company.com",
  "password": "manager123"
}
```

**Postman Script (Tests tab)**:
```javascript
if (pm.response.code === 200) {
    const response = pm.response.json();
    pm.environment.set("authToken", response.token);
    pm.environment.set("managerId", response._id);
}
```

#### Step 7: Get All Leaves (Manager)
```
GET {{baseUrl}}/leaves/all
Authorization: Bearer {{authToken}}
```

#### Step 8: Approve Leave (Manager)
```
PUT {{baseUrl}}/leaves/{{leaveId}}/approve
Authorization: Bearer {{authToken}}
```

#### Step 9: Reject Leave (Manager) - Alternative to approve
```
PUT {{baseUrl}}/leaves/{{leaveId}}/reject
Authorization: Bearer {{authToken}}
```

#### Step 10: Cancel Leave (Employee)
```
DELETE {{baseUrl}}/leaves/{{leaveId}}
Authorization: Bearer {{authToken}}
```

### Postman Collection Tests

#### Generic Test Script (for all requests)
```javascript
pm.test("Response time is less than 2000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});

pm.test("Response should be JSON", function () {
    pm.expect(pm.response.headers.get("Content-Type")).to.include("application/json");
});
```

#### Success Response Test
```javascript
pm.test("Status code is successful", function () {
    pm.expect(pm.response.code).to.be.oneOf([200, 201]);
});

pm.test("Response contains required fields", function () {
    const response = pm.response.json();
    if (response.message) {
        pm.expect(response.message).to.be.a('string');
    }
});
```

#### Role-Based Access Test
```javascript
pm.test("Forbidden access returns 403", function () {
    if (pm.response.code === 403) {
        pm.expect(pm.response.json().message).to.include('authorized');
    }
});
```

#### Leave Status Validation
```javascript
pm.test("Leave status is valid", function () {
    const response = pm.response.json();
    if (response.status) {
        pm.expect(response.status).to.be.oneOf(['pending', 'approved', 'rejected']);
    }
});
```

---

## ⚠️ Error Handling

### Common HTTP Status Codes

- **200 OK**: Successful GET, PUT, DELETE
- **201 Created**: Successful POST (registration, leave application)
- **400 Bad Request**: Invalid input, validation errors, business logic violations
- **401 Unauthorized**: Missing or invalid authentication token
- **403 Forbidden**: Insufficient permissions for the requested action
- **404 Not Found**: Resource not found
- **500 Internal Server Error**: Server-side errors

### Error Response Format

```json
{
  "message": "Error description"
}
```

---

## 🔍 Troubleshooting

### Common Issues

#### Authentication Issues
- **"Invalid credentials"**: Check email and password
- **"Not authorized"**: Include `Authorization: Bearer <token>` header
- **Token expired**: Re-login to get a new token

#### Authorization Issues
- **403 Forbidden**: User role doesn't have permission for this action
- **Employee accessing admin routes**: Only managers/admins can access certain routes
- **Accessing other user's data**: Employees can only access their own leave requests

#### Leave Management Issues
- **"Leave not found"**: Check leave ID format and existence
- **"Only pending leaves can be cancelled"**: Cannot cancel approved/rejected leaves
- **Invalid leave type**: Must be one of: sick, casual, earned
- **Invalid leave status**: Must be one of: pending, approved, rejected

#### Database Issues
- **Connection timeout**: Check MongoDB URI and connectivity
- **Validation errors**: Ensure all required fields are provided
- **Duplicate email**: Email already exists during registration

### Role Testing
Test different scenarios with different user roles:

1. **Employee Tests**:
   - Can apply for leaves ✅
   - Can view own leaves ✅
   - Cannot view all leaves ❌
   - Cannot approve/reject leaves ❌

2. **Manager Tests**:
   - Can do everything employees can ✅
   - Can view all leaves ✅
   - Can approve/reject leaves ✅

3. **Cross-User Tests**:
   - Employee A cannot cancel Employee B's leave ❌
   - Only the leave owner can cancel their pending leave ✅

---

## 📝 API Testing Checklist

### ✅ Authentication
- [ ] Register employee with valid data
- [ ] Register manager with valid data
- [ ] Register admin with valid data
- [ ] Register with existing email (should fail)
- [ ] Login with valid credentials
- [ ] Login with invalid credentials (should fail)

### ✅ Leave Management (Employee)
- [ ] Apply for sick leave
- [ ] Apply for casual leave
- [ ] Apply for earned leave
- [ ] Get own leave requests
- [ ] Cancel own pending leave
- [ ] Try to cancel approved leave (should fail)
- [ ] Try to access other's leaves (should fail)

### ✅ Leave Management (Manager/Admin)
- [ ] View all leave requests
- [ ] Approve pending leave request
- [ ] Reject pending leave request
- [ ] Try to approve already approved leave
- [ ] Try to reject already rejected leave

### ✅ Authorization & Security
- [ ] Access protected route without token (should fail)
- [ ] Access manager route as employee (should fail)
- [ ] Access admin route as manager (if applicable)
- [ ] Verify JWT token contains correct user info

### ✅ Business Logic
- [ ] Leave dates validation (start date before end date)
- [ ] Leave status transitions (pending → approved/rejected)
- [ ] Leave balance tracking (if implemented)
- [ ] Overlapping leave requests (if validation implemented)

---

## 🚀 Deployment

### Production Considerations

1. **Environment Variables**: Set secure values for production
2. **Database**: Use MongoDB Atlas or secured MongoDB instance
3. **HTTPS**: Enable SSL/TLS in production
4. **Rate Limiting**: Implement rate limiting middleware
5. **Logging**: Add comprehensive logging system
6. **Email Notifications**: Integrate email service for leave status updates
7. **Leave Balance Calculation**: Implement automatic leave balance deduction

### Additional Features for Production

1. **Email Notifications**: Notify employees and managers about leave status changes
2. **Leave Balance Management**: Automatic calculation and validation
3. **Leave Calendar**: Integration with calendar systems
4. **Reporting**: Generate leave reports and analytics
5. **Holiday Management**: Integration with company holiday calendar

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Ensure all tests pass
6. Submit a pull request

---

## 📄 License

This project is licensed under the MIT License.

---

## 📞 Support

For issues and questions:
- Create an issue in the repository
- Check the troubleshooting section
- Review error messages and status codes
- Test with different user roles to understand permission systems

---

**Happy Leave Management! 🏖️**#   G N C I P L - A S S I G N M E N T - W E E K - 4  
 