Leave Management System - Backend API

This project is a RESTful API for managing employee leaves. It is built with Node.js, Express.js, and MongoDB. The system provides authentication, role-based access, and leave management functionalities for employees, managers, and administrators.

Features

Authentication: JWT-based authentication with role-specific access.

Leave Management: Create, read, update, and delete leave requests.

Role Control: Employee, Manager, and Admin roles with distinct permissions.

Leave Types: Supports sick, casual, and earned leaves.

Status Tracking: Leave requests can be pending, approved, or rejected.

Leave Balance: Each employee has a default leave balance (20 days/year).

Table of Contents

Installation

Environment Variables

API Endpoints

Authentication

Leave Management

Manager/Admin

Data Models

Authentication & Authorization

Role-Based Permissions

Testing with Postman

Error Handling

Troubleshooting

Deployment

Contributing

License

Installation
Requirements

Node.js v14 or higher

MongoDB (local instance or MongoDB Atlas)

npm or yarn

Steps

Clone the repository:

git clone <repository-url>
cd assignment4


Install dependencies:

npm install


Copy the environment file:

cp .env.example .env


Update .env with your own values.

Start the server:

npm run dev   # development
npm start     # production


The API will run at: http://localhost:5000

Environment Variables

Create a .env file with the following keys:

PORT=5000
JWT_SECRET="your_secret_key"
MONGODB_URI="mongodb+srv://username:password@cluster.mongodb.net/leavemanagement"
# or local:
# MONGODB_URI="mongodb://localhost:27017/leavemanagement"
NODE_ENV="development"

API Endpoints
Base URL
http://localhost:5000/api

Authentication
Register User

POST /api/auth/register
Create a user account with a role.

Roles:

employee (default)

manager

admin

Login

POST /api/auth/login
Authenticate and receive a JWT token.

Leave Management
Apply for Leave

POST /api/leaves/apply
Submit a new leave request (employee only).

Get Own Leaves

GET /api/leaves/my
Retrieve all leave requests of the logged-in user.

Cancel Leave

DELETE /api/leaves/:id
Cancel a pending leave request (employees can only cancel their own).

Manager/Admin
Get All Leave Requests

GET /api/leaves/all
View all leave requests in the system.

Approve Leave

PUT /api/leaves/:id/approve
Approve a pending leave request.

Reject Leave

PUT /api/leaves/:id/reject
Reject a pending leave request.

Data Models
User
{
  _id: ObjectId,
  name: String,
  email: String (unique),
  password: String (hashed),
  role: "employee" | "manager" | "admin",
  leaveBalance: Number (default: 20),
  createdAt: Date,
  updatedAt: Date
}

Leave Request
{
  _id: ObjectId,
  employee: ObjectId (ref: User),
  leaveType: "sick" | "casual" | "earned",
  startDate: Date,
  endDate: Date,
  reason: String,
  status: "pending" | "approved" | "rejected",
  approvedBy: ObjectId (ref: User),
  createdAt: Date,
  updatedAt: Date
}

Authentication & Authorization

Uses JWT tokens for authentication.

Passwords are hashed using bcrypt.

Employees can only access their own data.

Managers/Admins can manage leave requests across the system.

Role-Based Permissions

Employee: Apply, view, and cancel own pending leaves.

Manager: All employee actions + approve/reject/view all leaves.

Admin: All manager actions + system-wide access and user management.

Testing with Postman

Suggested test flow:

Register employee and manager.

Login as employee → Apply for leave.

Login as manager → View and approve/reject leave.

Employee attempts to cancel approved/rejected leave (should fail).

Check role-based restrictions.

Use environment variables in Postman (baseUrl, authToken, leaveId) for convenience.

Error Handling

200/201: Success

400: Validation error

401: Unauthorized (missing/invalid token)

403: Forbidden (insufficient role)

404: Not found

500: Server error

Response format:

{ "message": "Error description" }

Troubleshooting

Invalid credentials: Check email/password.

Unauthorized: Ensure token is sent in the Authorization header.

Forbidden: Role doesn’t have access to the route.

Leave not found: Invalid or missing leave ID.

MongoDB errors: Verify connection string and database availability.

Deployment

For production:

Use secure environment variables.

Host MongoDB on Atlas or a secured server.

Enable HTTPS.

Consider rate limiting and logging.

Optionally add email notifications for leave status updates.

Contributing

Fork the repo

Create a branch

Implement changes with tests

Submit a pull request

License

This project is available under the MIT License.