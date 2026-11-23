# MongoDB Backend API

A REST and real-time backend built with Node.js, Express, MongoDB/Mongoose, JWT authentication, and Socket.IO.

The API supports user registration and login, profile management, password reset emails, role-based admin operations, product CRUD operations, and real-time chat.

## Features

- User registration and login with JWT cookies
- Password reset with email and short-lived reset tokens
- Authenticated user profile retrieval and updates
- Admin endpoints for listing, updating, and deleting users
- Product listing, searching, and creation
- Role-based authorization
- Public chat messages and private direct messages over Socket.IO
- Request validation with Joi
- Password hashing with bcrypt

## Tech Stack

- Node.js with ES modules
- Express 5
- MongoDB and Mongoose
- JSON Web Tokens
- Socket.IO
- Joi
- bcrypt
- Nodemailer
- MailDev
- CORS and cookie-parser

## Project Structure

```text
.
|-- scr/
|   |-- controllers/
|   |-- middleware/
|   |-- models/
|   |-- routes/
|   |-- services/
|   `-- validators/
|-- scripts/
|   `-- remove-unique-role-index.js
|-- socket/
|   `-- chatSocket.js
|-- server.js
|-- package.json
`-- README.md
```

## Prerequisites

- Node.js 18 or newer
- npm
- MongoDB, local or hosted
- MailDev for local email testing

## Installation

```bash
npm install
```

Create a `.env` file in the project root:

```env
PORT=3000
DATABASE_URL=mongodb://127.0.0.1:27017/your_database_name
SECRET=replace-with-a-long-random-secret
```

- `PORT` is the HTTP and Socket.IO server port.
- `DATABASE_URL` is the MongoDB connection string.
- `SECRET` signs and verifies JWT access and password-reset tokens.

Never commit `.env` or real credentials to GitHub.

## Running the Project

Start the API in development mode:

```bash
npm run dev
```

For a non-watching production-style process:

```bash
npm start
```

The HTTP server listens on `127.0.0.1` and is available at:

```text
http://127.0.0.1:3000
```

Update the port in the URL if `PORT` is changed.

Start the local MailDev server for welcome and password-reset emails:

```bash
npx maildev
```

MailDev commonly exposes its web interface at `http://127.0.0.1:1080` and its SMTP server on port `1025`.

## API Routes

### Authentication

| Method | Route | Description |
| --- | --- | --- |
| `POST` | `/register` | Register a user |
| `POST` | `/login` | Authenticate a user and set the JWT cookie |
| `POST` | `/auth/forgotpassword` | Request a password-reset email |
| `POST` | `/auth/confirmtoken` | Validate a password-reset token |
| `POST` | `/auth/resetpassword` | Reset a password |

### Users

| Method | Route | Description |
| --- | --- | --- |
| `GET` | `/user/:userName` | Get an authenticated user's profile |
| `PATCH` | `/user/:userName` | Update an authenticated user's profile |
| `DELETE` | `/user/:userName` | Delete an authenticated user |

### Admin

All admin routes require a valid authenticated account with the `admin` role.

| Method | Route | Description |
| --- | --- | --- |
| `GET` | `/admin/users` | Get all users |
| `PATCH` | `/admin/users/:userId/role` | Update a user's role |
| `DELETE` | `/admin/users/:userId` | Delete a user |

### Products

| Method | Route | Description |
| --- | --- | --- |
| `GET` | `/products` | Get all products |
| `GET` | `/products/:name` | Get a product by name |
| `POST` | `/products` | Create a product |

## Authentication

JWTs are stored as an HTTP-only `token` cookie. The login and registration endpoints set this cookie. Protected HTTP routes read it from cookies or the `Cookie` header.

Socket.IO connections authenticate through the same cookie. The server has no public email/password socket login flow.

## Registration Example

```http
POST /register
Content-Type: application/json

{
  "userName": "johndoe",
  "fullName": "John Doe",
  "email": "john@example.com",
  "password": "strongpassword",
  "phone": "12345678901",
  "gender": "male",
  "address": "Example Address",
  "age": 25
}
```

## Socket.IO Events

The Socket.IO server uses the same host and port as the Express server.

### Server-sent events

- `active user` - current active users
- `chat message` - a public message
- `private message` - a direct message for the recipient

### Client-sent events

- `chat message` - send a public text message
- `private message` - send a direct message

Private message payload:

```json
{
  "recipientId": "mongodb-user-object-id",
  "text": "Hello!"
}
```

The recipient ID must be a valid MongoDB 24-character ObjectId, and message text is limited to 2000 characters.

## Database Migration

This project contains a migration that removes an obsolete unique `role` index from the `users` collection:

```bash
npm run migrate:role-index
```

The migration requires `DATABASE_URL` in `.env`.

## CORS

The API currently allows CORS only for:

```text
http://127.0.0.1:5500
```

Update the origin in `scr/app.js` when serving the frontend from another host or port.

## Security Notes

- Use a strong, unique `SECRET` in production.
- Use HTTPS and secure cookie settings behind a production deployment.
- Restrict MongoDB network access and database credentials.
- Add production email and frontend URL configuration instead of relying on the hard-coded local development URLs.
- Do not expose JWTs or database credentials in logs or client-side code.

## License

ISC
