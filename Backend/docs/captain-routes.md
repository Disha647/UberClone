# Captain routes — API documentation

Base path: `/captains`

---

## POST /captains/register  ✅
Register a new captain (driver). Returns an auth token and the created captain record.

### Auth
- Public (no token required)

### Request headers
- `Content-Type: application/json`

### Request body (JSON)
Required fields are validated by `express-validator` and by the Mongoose model.

```json
{
  "fullname": { "firstname": "John", "lastname": "Doe" },
  "email": "john@example.com",
  "password": "secret123",
  "vehicle": {
    "color": "white",
    "plate": "AB1234",
    "capacity": 4,
    "vehicleType": "car"
  }
}
```

### Validation rules
- `email` — must be a valid email
- `fullname.firstname` — string, min length 3
- `password` — string, min length 6
- `vehicle.color` — string, min length 3
- `vehicle.plate` — string, min length 3
- `vehicle.capacity` — integer, min 1
- `vehicle.vehicleType` — one of `car`, `motorcycle`, `auto`

> Note: `fullname.lastname` is optional but recommended.

### Successful response (201 Created)
Content-Type: `application/json`

Body:
```json
{
  "token": "<jwt-token>",
  "captain": {
    "_id": "603d...",
    "fullname": { "firstname": "John", "lastname": "Doe" },
    "email": "john@example.com",
    "status": "inactive",
    "vehicle": { "color": "white", "plate": "AB1234", "capacity": 4, "vehicleType": "car" },
    "location": { "ltd": null, "lng": null },
    "socketId": null
  }
}
```

Security note: the current implementation returns the created `captain` document as-is. Ensure you do NOT expose the password hash in API responses (see "Best practice" below).

### Error responses
- 400 Bad Request — validation failed
  - Response shape from `express-validator`:
    ```json
    { "errors": [ { "msg": "Invalid Email", "param": "email", "location": "body", "value": "foo" } ] }
    ```
- 400 Bad Request — duplicate email
  - `{ "message": "Captain already exist" }`
- 500 Internal Server Error — unexpected errors

### Example cURL
```bash
curl -X POST 'http://localhost:3000/captains/register' \
  -H 'Content-Type: application/json' \
  -d '{
    "fullname": { "firstname": "Jane", "lastname": "Roe" },
    "email": "jane@example.com",
    "password": "password123",
    "vehicle": { "color": "red", "plate": "XYZ-100", "capacity": 4, "vehicleType": "car" }
  }'
```

### Best practice / Implementation tip
- Do not return password hashes in responses. Filter the `captain` object before sending (e.g. `captain = captain.toObject(); delete captain.password;`).

Suggested change in `registerCaptain` (example):
```js
const created = await captainService.createCaptain(...);
const captainObj = created.toObject();
delete captainObj.password;
res.status(201).json({ token, captain: captainObj });
```

### Related files
- `routes/captain.routes.js`
- `controllers/captain.controller.js`
- `models/captain.model.js`

---

If you want, I can:
1. Add examples for error responses in `README.md` ✅
2. Update controller to strip `password` from the response 🔧
3. Create Postman collection for the captain routes 💡

Which of these should I do next?