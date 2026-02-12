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

