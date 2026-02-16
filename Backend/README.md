# User Registration — `POST /user/register`

## Description
Registers a new user. The endpoint validates input, hashes the password, creates the user record, and returns a JWT plus the created user (password is not returned).

---

## Endpoint
- URL: `/user/register`
- Method: `POST`
- Content-Type: `application/json`

## Request body
JSON object with the following structure:

{
  "fullname": {
    "firstname": "string",    // required, min length 3
    "lastname": "string"      // optional, min length 3
  },
  "email": "string",         // required, must be a valid email
  "password": "string"       // required, min length 6
}

Validation rules
- `fullname.firstname`: required, minimum 3 characters
- `fullname.lastname`: optional, if provided minimum 3 characters
- `email`: required, must be valid email format
- `password`: required, minimum 6 characters

## Success response
- Status: `201 Created`
- Body: `{ token, user }`
  - `token`: JWT string (signed with `process.env.JWT_SECRET`)
  - `user`: created user document (password excluded)

Example success response

{
  "token": "<jwt-token>",
  "user": {
    "_id": "632...",
    "fullname": { "firstname": "Disha", "lastname": "Patel" },
    "email": "disha@example.com",
    "socketId": null,
    "__v": 0
  }
}

## Error responses
- `400 Bad Request` — validation failed
  - Body shape: `{ errors: [ { msg, param, location, value } ] }`

Example validation error

{
  "errors": [
    { "msg": "Password must be at least 6 characters long", "param": "password", "location": "body" }
  ]
}

- `500 Internal Server Error` — unexpected server error

## Notes / Implementation details
- Passwords are hashed before storage.
- The JWT is generated using `user.generateAuthToken()` and requires `JWT_SECRET` in environment.
- Returned `user` object excludes the password field (model `select: false`).

## Example curl

curl -X POST http://localhost:3000/user/register \
  -H "Content-Type: application/json" \
  -d '{"fullname": {"firstname":"Disha","lastname":"Patel"}, "email":"disha@example.com", "password":"s3cret"}'

---

# User Login — `POST /user/login`

## Description
Authenticates an existing user. Validates credentials and returns a JWT plus the user object (password excluded).

## Endpoint
- URL: `/user/login`
- Method: `POST`
- Content-Type: `application/json`

## Request body
JSON object with the following structure:

{
  "email": "string",      // required, must be a valid email
  "password": "string"    // required, min length 6
}

Validation rules
- `email`: required, must be a valid email
- `password`: required, minimum 6 characters

## Success response
- Status: `200 OK`
- Body: `{ token, user }`
  - `token`: JWT string
  - `user`: user document (password excluded)

Example success response

{
  "token": "<jwt-token>",
  "user": {
    "_id": "632...",
    "fullname": { "firstname": "Disha", "lastname": "Patel" },
    "email": "disha@example.com",
    "socketId": null,
    "__v": 0
  }
}

## Error responses
- `400 Bad Request` — validation failed (returns `{ errors: [...] }`)
- `401 Unauthorized` — invalid credentials
  - Body: `{ message: "Invalid email or password" }`
- `500 Internal Server Error` — unexpected server error

Example invalid-credentials response

{
  "message": "Invalid email or password"
}

## Example curl

curl -X POST http://localhost:3000/user/login \
  -H "Content-Type: application/json" \
  -d '{"email":"disha@example.com", "password":"s3cret"}'

---

# User Profile — `GET /users/profile`

## Description
Returns the authenticated user's profile. The endpoint is protected — a valid JWT must be supplied (cookie or Authorization header). The response contains the user document with the password excluded.

---

## Endpoint
- URL: `/users/profile`
- Method: `GET`
- Authentication: Required (JWT)

## Authentication
Provide the JWT either as:
- `Authorization: Bearer <token>` header
- or a cookie named `token`

## Success response
- Status: `200 OK`
- Body: the authenticated `user` object (password is not returned)

Example success response

{
  "_id": "632...",
  "fullname": { "firstname": "Disha", "lastname": "Patel" },
  "email": "disha@example.com",
  "socketId": null,
  "__v": 0
}

## Error responses
- `401 Unauthorized` — missing/invalid/blacklisted token
  - Body: `{ message: "Unauthorized" }`
- `500 Internal Server Error` — unexpected server error

## Example curl (Authorization header)

curl -X GET http://localhost:3000/users/profile \
  -H "Authorization: Bearer <jwt-token>"

## Example curl (cookie)

curl -X GET http://localhost:3000/users/profile \
  --cookie "token=<jwt-token>"

---

# User Logout — `GET /users/logout`

## Description
Logs out the authenticated user by clearing the `token` cookie and adding the token to a server-side blacklist (prevents reuse until the token expires).

---

## Endpoint
- URL: `/users/logout`
- Method: `GET`
- Authentication: Required (JWT)

## Behavior
- Clears the `token` cookie on the client (`res.clearCookie('token')`).
- Stores the token in the `BlacklistToken` collection (expires after 24 hours).
- Returns a confirmation message.

## Success response
- Status: `200 OK`
- Body: `{ "message": "Logged out" }`

## Error responses
- `401 Unauthorized` — missing/invalid/blacklisted token
  - Body: `{ message: "Unauthorized" }`
- `500 Internal Server Error` — unexpected server error

## Example curl (Authorization header)

curl -X GET http://localhost:3000/users/logout \
  -H "Authorization: Bearer <jwt-token>"

## Example curl (cookie)

curl -X GET http://localhost:3000/users/logout \
  --cookie "token=<jwt-token>"

---

# Captain Registration — `POST /captains/register`

## Description
Registers a new captain (driver). Validates input, hashes the password, creates the captain record, and returns a JWT plus the created captain (password is not returned).

---

## Endpoint
- URL: `/captains/register`
- Method: `POST`
- Content-Type: `application/json`

## Request body
JSON object with the following structure:

{
  "fullname": {
    "firstname": "string",    // required, min length 3
    "lastname": "string"      // optional, min length 3
  },
  "email": "string",         // required, must be a valid email
  "password": "string",      // required, min length 6
  "vehicle": {
    "color": "string",       // required, min length 3
    "plate": "string",       // required, min length 3
    "capacity": number,        // required, min 1
    "vehicleType": "string"  // required, one of: car, motorcycle, auto
  }
}

### Validation rules
- `email`: required, valid email
- `fullname.firstname`: required, minimum 3 characters
- `password`: required, minimum 6 characters
- `vehicle.color`: required, minimum 3 characters
- `vehicle.plate`: required, minimum 3 characters
- `vehicle.capacity`: integer, minimum 1
- `vehicle.vehicleType`: one of `car`, `motorcycle`, `auto`

## Success response
- Status: `201 Created`
- Body: `{ token, captain }`
  - `token`: JWT string (signed with `process.env.JWT_SECRET`)
  - `captain`: created captain document (password excluded)

Example success response

{
  "token": "<jwt-token>",
  "captain": {
    "_id": "603...",
    "fullname": { "firstname": "John", "lastname": "Doe" },
    "email": "john@example.com",
    "status": "inactive",
    "vehicle": { "color": "white", "plate": "AB1234", "capacity": 4, "vehicleType": "car" },
    "location": { "ltd": null, "lng": null },
    "socketId": null
  }
}

## Error responses
- `400 Bad Request` — validation failed (returns `{ errors: [...] }` from `express-validator`)
- `400 Bad Request` — duplicate email (`{ "message": "Captain already exist" }`)
- `500 Internal Server Error` — unexpected server error

## Example curl

curl -X POST http://localhost:3000/captains/register \
  -H "Content-Type: application/json" \
  -d '{
    "fullname": { "firstname": "Jane", "lastname": "Roe" },
    "email": "jane@example.com",
    "password": "password123",
    "vehicle": { "color": "red", "plate": "XYZ-100", "capacity": 4, "vehicleType": "car" }
  }'


---

