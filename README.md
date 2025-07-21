# Payments API Route Documentation

## General

### `GET /<version>/healthcheck`
- **Description:** Health check endpoint.
- **Auth:** None
- **Input:** None
- **Status Codes:**
  - `200 OK` – Service is healthy

---

### `GET /<version>/settings`
- **Description:** Returns app settings, splash screen, onboarding, and useful links.
- **Auth:** None
- **Input:** None
- **Status Codes:**
  - `200 OK` – Success
  - `404 Not Found` – Settings not found

---

### `GET /<version>/public/*filepath`
- **Description:** Serves static files (e.g., images).
- **Auth:** None
- **Input:** Path parameter for static file.
- **Status Codes:**
  - `200 OK` – File found
  - `404 Not Found` – File not found

---

## Country

### `GET /<version>/countries/:country_code`
- **Description:** Returns exchange rates and providers for the specified country.
- **Auth:** None
- **Input:** Path parameter: `country_code` (required)
- **Status Codes:**
  - `200 OK` – Success
  - `404 Not Found` – Country not found

---

## Payments

### `POST /<version>/payments/submit`
- **Description:** Submits a payment using temporary stored data.
- **Auth:** Requires activated user
- **Body:**
  ```json
  {
    "identifier": "string" // required, temporary payment data identifier
  }
  ```
- **Status Codes:**
  - `201 Created` – Payment submitted
  - `400 Bad Request` – Invalid input
  - `401 Unauthorized` – Not authenticated/activated
  - `404 Not Found` – Temporary data not found
  - `422 Unprocessable Entity` – Validation error

---

### `GET /<version>/payments/types`
- **Description:** Returns supported countries and payout method settings.
- **Auth:** Requires activated user
- **Input:** None
- **Status Codes:**
  - `200 OK` – Success
  - `401 Unauthorized` – Not authenticated/activated

---

### `POST /<version>/payments/store`
- **Description:** Stores temporary payment data.
- **Auth:** Requires activated user
- **Body:**
  ```json
  {
    "type": "string",              // required, payment method type (e.g., bank transfer)
    "send_money": 123,             // required, amount to send
    "sender_currency": "string",   // required, sender country ID
    "receiver_currency": "string", // required, receiver country ID
    "coupon": "string",            // optional
    "coupon_type": "string"        // optional
  }
  ```
- **Status Codes:**
  - `201 Created` – Data stored
  - `400 Bad Request` – Invalid input
  - `401 Unauthorized` – Not authenticated/activated
  - `404 Not Found` – Country not found
  - `422 Unprocessable Entity` – Validation error

---

### `PUT /<version>/payments/store/recipients`
- **Description:** Updates temporary payment data with recipient info.
- **Auth:** Requires activated user
- **Body:**
  ```json
  {
    "beneficiary_id": "string", // required, recipient ID
    "identifier": "string"      // required, temporary data ID
  }
  ```
- **Status Codes:**
  - `200 OK` – Update successful
  - `400 Bad Request` – Invalid input
  - `401 Unauthorized` – Not authenticated/activated
  - `404 Not Found` – Data or recipient not found
  - `422 Unprocessable Entity` – Validation error

---

### `GET /<version>/payments/users/:provider_payment_id`
- **Description:** Gets payment details by ID.
- **Auth:** Requires activated user
- **Input:** Path parameter: `provider_payment_id` (required)
- **Status Codes:**
  - `200 OK` – Success
  - `401 Unauthorized` – Not authenticated/activated
  - `404 Not Found` – Payment not found

---

### `GET /<version>/payments/recipients/:identifier`
- **Description:** Gets available recipients for a user from a specific country.
- **Auth:** Requires activated user
- **Input:** Path parameter: `identifier` (required, temporary data ID)
- **Status Codes:**
  - `200 OK` – Success (empty list if none)
  - `401 Unauthorized` – Not authenticated/activated
  - `404 Not Found` – Data not found

---

### `POST /<version>/payments/recipients/create`
- **Description:** Creates a recipient for a country.
- **Auth:** Requires activated user
- **Body:**
  ```json
  {
    "first_name": "string",         // required
    "last_name": "string",          // required
    "email": "string",              // required
    "country": "string",            // required if country_id not provided
    "country_id": "string",         // required if country not provided
    "payout_method_id": "string",   // required
    "payout_method": "string",      // required
    "account_number": "string",     // required for bank/mobile payouts
    "address": {
      "street": "string",           // optional
      "city": "string",             // optional
      "state": "string",            // optional
      "zip_code": "string",         // optional
      "country": "string"           // optional, can be used as country
    },
    "bank_address": { ... },        // optional, for bank payouts
    "middle_name": "string",        // optional
    "phone": "string",              // optional
    "mobile_name": "string",        // optional, for mobile money
    "bank_name": "string",          // optional
    "iban_number": "string",        // optional
    "document_type": "string",      // optional
    "front_image": "string",        // optional
    "back_image": "string"          // optional
  }
  ```
  - **At least one of** `country` or `country_id` is required.
  - **At least one of** `country`, `address.country`, or `bank_address.country` must be provided if `country_id` is not set.
- **Status Codes:**
  - `201 Created` – Recipient created
  - `400 Bad Request` – Invalid input
  - `401 Unauthorized` – Not authenticated/activated
  - `404 Not Found` – Country not found
  - `422 Unprocessable Entity` – Validation error

---

### `GET /<version>/payments/payout-methods`
- **Description:** Gets payout methods for a specific country.
- **Auth:** Requires activated user
- **Input:** None
- **Status Codes:**
  - `200 OK` – Success
  - `401 Unauthorized` – Not authenticated/activated

---

### `GET /<version>/payments/source-purpose`
- **Description:** Gets sending purposes and source of funds.
- **Auth:** Requires activated user
- **Input:** None
- **Status Codes:**
  - `200 OK` – Success
  - `401 Unauthorized` – Not authenticated/activated

---

### `POST /<version>/payments/store/source-purpose`
- **Description:** Stores purpose of funds in temporary store.
- **Auth:** Requires activated user
- **Body:**
  ```json
  {
    "identifier": "string",        // required, temporary data ID
    "sending_purpose": "string",   // required, sending purpose ID
    "source": "string",            // required, source of funds ID
    "remark": "string",            // optional
    "payment_gateway": "string"    // required, payment gateway ID
  }
  ```
- **Status Codes:**
  - `200 OK` – Update successful
  - `400 Bad Request` – Invalid input
  - `401 Unauthorized` – Not authenticated/activated
  - `404 Not Found` – Data not found
  - `422 Unprocessable Entity` – Validation error

---

## Webhook

### `POST /<version>/webhook`
- **Description:** Handles incoming webhooks from payment/payout providers.
- **Auth:** None
- **Input:** Raw body, with provider-specific headers (e.g., `Stripe-Signature`, `Signature`, `PayPal-Auth-Assertion`)
- **Status Codes:**
  - `200 OK` – Webhook processed or acknowledged
  - `400 Bad Request` – Invalid input
  - `500 Internal Server Error` – Processing error

---

## User

### `POST /<version>/user/register`
- **Description:** Registers a new user.
- **Auth:** None
- **Body:**
  ```json
  {
    "first_name": "string",        // required
    "last_name": "string",         // required
    "email": "string",             // required, must be valid email
    "password": "string",          // required, must meet password policy
    "date_of_birth": "YYYY-MM-DD", // optional
    "phone": "string",             // optional
    "country": "string",           // required if address.country not provided
    "policy": "string",            // optional
    "address": {
      "street": "string",          // optional
      "city": "string",            // optional
      "state": "string",           // optional
      "zip_code": "string",        // optional
      "country": "string"          // required if country not provided
    }
  }
  ```
  - **At least one of** `country` or `address.country` is required.
- **Status Codes:**
  - `201 Created` – User registered
  - `400 Bad Request` – Invalid input
  - `422 Unprocessable Entity` – Validation error

---

### `POST /<version>/user/email/code/activate`
- **Description:** Activates a user account with an OTP code.
- **Auth:** Requires authenticated user
- **Body:**
  ```json
  {
    "code": "string" // required, OTP code
  }
  ```
- **Status Codes:**
  - `200 OK` – Activation successful
  - `400 Bad Request` – Invalid input
  - `401 Unauthorized` – Not authenticated
  - `422 Unprocessable Entity` – Validation error

---

### `POST /<version>/user/logout`
- **Description:** Logs out the user.
- **Auth:** Requires authenticated user
- **Input:** None
- **Status Codes:**
  - `200 OK` – Logout successful
  - `401 Unauthorized` – Not authenticated

---

### `GET /<version>/user`
- **Description:** Gets user details.
- **Auth:** Requires authenticated user
- **Input:** None
- **Status Codes:**
  - `200 OK` – Success
  - `401 Unauthorized` – Not authenticated

---

### `GET /<version>/user/profile`
- **Description:** Gets user profile information.
- **Auth:** Requires authenticated user
- **Input:** None
- **Status Codes:**
  - `200 OK` – Success
  - `401 Unauthorized` – Not authenticated

---

### `GET /<version>/user/notifications`
- **Description:** Gets user notifications.
- **Auth:** Requires authenticated user
- **Input:** None
- **Status Codes:**
  - `200 OK` – Success
  - `401 Unauthorized` – Not authenticated

---

## Token

### `POST /<version>/token/auth`
- **Description:** Authenticates a user and returns a token.
- **Auth:** None
- **Body:**
  ```json
  {
    "email": "string",    // required, must be valid email
    "password": "string"  // required
  }
  ```
- **Status Codes:**
  - `200 OK` – Authenticated, token returned
  - `400 Bad Request` – Invalid input
  - `401 Unauthorized` – Invalid credentials
  - `422 Unprocessable Entity` – Validation error

---

## Notes for UI/Frontend Team

- All endpoints with `Auth: Requires authenticated user` require a valid Bearer token in the `Authorization` header.
- All endpoints with `Auth: Requires activated user` require the user to be both authenticated and activated.
- All required fields must be present and non-empty unless otherwise specified.
- For endpoints with multiple ways to specify a required field (e.g., `country` or `address.country`), at least one must be provided.
- Validation errors will return a `422` status with details in the response.
- `400` is used for malformed or missing input, `404` for not found, `401` for authentication/authorization issues, and `500` for server errors.

---

**If you need example responses, error formats, or more details on any endpoint, let me know!** 