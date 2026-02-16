# Captain routes — API documentation

Base path: `/captains`

---

This file documents every route mounted under `/captains` (register, login, profile, logout). Request bodies and responses are shown as JSON with inline comments describing validation and requirements.

## POST /captains/register  ✅
Create a new captain (driver). Returns a JWT and the created captain resource.

- Auth: Public
- Content-Type: `application/json`

Request body (JSON with comments):
```json
{
  "fullname": {
    "firstname": "John",    // required, string, min length 3
    "lastname": "Doe"       // optional, string, min length 3
  },
  "email": "john@example.com", // required, valid email
  "password": "secret123",     // required, min length 6
  "vehicle": {
    "color": "white",          // required, min length 3
    "plate": "AB1234",         // required, min length 3
    "capacity": 4,               // required, integer >= 1
    "vehicleType": "car"       // required, one of: "car", "motorcycle", "auto"
  }
}
```

Success response (201 Created):
```json
{
  "token": "<jwt-token>",
  "captain": {
    "_id": "603d...",
    "fullname": { "firstname": "John", "lastname": "Doe" },
    "email": "john@example.com",
    "status": "inactive",                              // default value from model
    "vehicle": { "color": "white", "plate": "AB1234", "capacity": 4, "vehicleType": "car" },
    "location": { "ltd": null, "lng": null },
    "socketId": null,
    "__v": 0
    // password is intentionally excluded from the response
  }
}
```

Errors:
- 400 — validation errors (returns `errors: []` from `express-validator`)
- 400 — duplicate email: `{ "message": "Captain already exist" }`
- 500 — server error

---

## POST /captains/login  🔐
Authenticate a captain and issue a JWT. Successful login also sets a `token` cookie.

- Auth: Public
- Content-Type: `application/json`

Request body (JSON with comments):
```json
{
  "email": "john@example.com", // required, valid email
  "password": "secret123"      // required, min length 6
}
```

Success response (200 OK):
```json
{
  "token": "<jwt-token>",
  "captain": {
    "_id": "603d...",
    "fullname": { "firstname": "John", "lastname": "Doe" },
    "email": "john@example.com",
    "status": "active",
    "vehicle": { "color": "white", "plate": "AB1234", "capacity": 4, "vehicleType": "car" }
    // password is excluded
  }
}
```

Errors:
- 400 — validation failed (`{ errors: [...] }`)
- 401 — invalid credentials: `{ "message": "Invalid email or password" }`

---

## GET /captains/profile  🧾
Return the authenticated captain's profile.

- Auth: Required (JWT via cookie or `Authorization: Bearer <token>`)

Success response (200 OK):
```json
{
  "captain": {
    "_id": "603d...",
    "fullname": { "firstname": "John", "lastname": "Doe" },
    "email": "john@example.com",
    "status": "active",
    "vehicle": { "color": "white", "plate": "AB1234", "capacity": 4, "vehicleType": "car" },
    "location": { "ltd": 12.34, "lng": 56.78 }
    // password is excluded
  }
}
```

Errors:
- 401 — missing/invalid/blacklisted token (`{ "message": "Unauthorized" }`)

---

## GET /captains/logout  🚪
Invalidate the current JWT (adds the token to `BlacklistToken`) and clears the cookie.

- Auth: Required (JWT)

Success response (200 OK):
```json
{ "message": "Logout successfully" }
```

Errors:
- 401 — missing/invalid/blacklisted token

---

### Validation summary
- Implemented via `express-validator` in `routes/captain.routes.js`.
- Model-level constraints enforced by `models/captain.model.js` (minlength, enum, required, etc.).

### Security note
- Controllers already hash passwords before save; do NOT return `password` in API responses.

### Related files
- `routes/captain.routes.js`
- `controllers/captain.controller.js`
- `models/captain.model.js`

---
