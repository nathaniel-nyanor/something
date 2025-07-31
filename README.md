Welcome to the Mini E-commerce API! These endpoints allow customers to browse products, register an account, buy as a guest, manage their shopping cart, place orders, and securely pay using Paystack.

> **Base URL:**  
> `https://api.mini-ecommerce.com/v1`

## Table of Contents

- [Authentication & Users](#authentication--users)
    - [Register](#register)
    - [Login](#login)
- [Guest Checkout](#guest-checkout)
- [Products](#products)
    - [Get All Products](#get-all-products)
    - [Get Product Details](#get-product-details)
- [Cart](#cart)
    - [Add Item to Cart](#add-item-to-cart)
    - [View Cart](#view-cart)
    - [Remove Item from Cart](#remove-item-from-cart)
- [Orders](#orders)
    - [Place Order (Authenticated)](#place-order-authenticated)
    - [Place Order (Guest)](#place-order-guest)
- [Payments (Paystack)](#payments-paystack)
    - [Initialize Payment](#initialize-payment)
    - [Verify Payment](#verify-payment)
- [Error Codes](#error-codes)

## Authentication & Users

### Register

**POST** `/register`

Registers a new user.

**Request Body**
| Field    | Type   | Required | Description               |
|----------|--------|----------|---------------------------|
| name     | string | Yes      | Full name of user         |
| email    | string | Yes      | Email address (unique)    |
| password | string | Yes      | Min 6 characters          |

```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "secret123"
}
```

**Response**  
_201 Created_
```json
{
  "userId": "u24j8fs92k",
  "name": "Jane Doe",
  "email": "jane@example.com"
}
```

### Login

**POST** `/login`

Authenticate user and retrieve token.

**Request Body**
| Field    | Type   | Required | Description      |
|----------|--------|----------|------------------|
| email    | string | Yes      | User email       |
| password | string | Yes      | User password    |

```json
{
  "email": "jane@example.com",
  "password": "secret123"
}
```

**Response**  
_200 OK_
```json
{
  "token": "eyJhbGciOiJIUzI1Ni...",
  "userId": "u24j8fs92k"
}
```

## Guest Checkout

Guest users can place orders without registering. A temporary guest account is created for the transaction.

**POST** `/guest/checkout`

**Request Body**
| Field     | Type   | Required | Description              |
|-----------|--------|----------|--------------------------|
| name      | string | Yes      | Full name (for shipping) |
| email     | string | Yes      | Contact email address    |
| cart      | array  | Yes      | Array of cart items      |

_Cart item schema:_
```json
{
  "productId": "p123",
  "quantity": 2
}
```

**Example**
```json
{
  "name": "Guest Buyer",
  "email": "guest@example.com",
  "cart": [
    { "productId": "p123", "quantity": 2 }
  ]
}
```

**Response**  
_201 Created_
```json
{
  "orderId": "o890",
  "total": 159998,
  "status": "pending_payment",
  "customer": {
      "name": "Guest Buyer",
      "email": "guest@example.com"
  }
}
```

## Products

### Get All Products

**GET** `/products`

Retrieve product catalog.

**Response** _200 OK_
```json
[
  {
    "id": "p123",
    "name": "Smartphone X2",
    "price": 79999,
    "description": "A next-gen smartphone.",
    "inStock": true
  }
]
```

### Get Product Details

**GET** `/products/:id`

**Response** _200 OK_
```json
{
  "id": "p123",
  "name": "Smartphone X2",
  "price": 79999,
  "description": "A next-gen smartphone.",
  "inStock": true
}
```

## Cart

_Authenticated users only._

### Add Item to Cart

**POST** `/cart`

| Field     | Type   | Required | Description       |
|-----------|--------|----------|-------------------|
| productId | string | Yes      | Product to add    |
| quantity  | int    | Yes      | How many?         |

```json
{
  "productId": "p123",
  "quantity": 1
}
```

**Response**  
_200 OK_
```json
{
  "message": "Added to cart",
  "cart": [{ "productId": "p123", "quantity": 1 }]
}
```

### View Cart

**GET** `/cart`

**Response**
```json
[ { "productId": "p123", "name": "Smartphone X2", "quantity": 1, "price": 79999 } ]
```

### Remove Item from Cart

**DELETE** `/cart/:itemId`

**Response**
```json
{ "message": "Removed from cart" }
```

## Orders

### Place Order (Authenticated)

**POST** `/orders`

_No request body needed; places an order from current cart._

**Response**
```json
{
  "orderId": "o890",
  "items": [ { "productId": "p123", "quantity": 1 } ],
  "total": 79999,
  "status": "pending_payment"
}
```

### Place Order (Guest)

See [Guest Checkout](#guest-checkout).  
Guest order can be placed with `/guest/checkout` and can proceed to payment step similarly.

## Payments (Paystack)

Payments are made via Paystack after placing an order (either as registered user or guest).

### Initialize Payment

**POST** `/payments/paystack/initialize`

_Provide the order ID and user or guest email. Amount is in kobo (smallest NGN unit)._

| Field    | Type   | Required | Description              |
|----------|--------|----------|--------------------------|
| email    | string | Yes      | Customer email           |
| orderId  | string | Yes      | Order to pay for         |
| amount   | int    | Yes      | Total in kobo            |

```json
{
  "email": "guest@example.com",
  "orderId": "o890",
  "amount": 159998
}
```

**Response**  
_200 OK_
```json
{
  "status": true,
  "message": "Authorization URL created",
  "data": {
    "authorization_url": "https://checkout.paystack.com/example1234",
    "access_code": "example1234",
    "reference": "xyzpaystackref"
  }
}
```

Customer should be redirected to `authorization_url` to make payment.

### Verify Payment

**GET** `/payments/paystack/verify/:reference`

| Parameter  | Type   | Description                         |
|------------|--------|-------------------------------------|
| reference  | string | Paystack transaction reference      |

Example:
```
GET /payments/paystack/verify/xyzpaystackref
```

**Response**  
_200 OK_
```json
{
  "status": true,
  "message": "Verification successful",
  "data": {
    "amount": 159998,
    "currency": "NGN",
    "status": "success",
    "paid_at": "2025-07-31T10:00:00Z",
    "customer": { "email": "guest@example.com" }
  }
}
```
Order status will be marked as "paid" upon successful verification.

## Error Codes

| Status | Meaning              | Typical Reason                             |
|--------|----------------------|--------------------------------------------|
| 400    | Bad Request          | Invalid inputs, missing fields             |
| 401    | Unauthorized         | Missing/invalid authentication             |
| 404    | Not Found            | Resource does not exist                    |
| 402    | Payment Required     | Attempted unpaid or failed payments        |
| 409    | Conflict             | Email already taken; data conflict         |
| 500    | Internal Server Error| Server or 3rd-party payment error          |

**Error Example**
```json
{
  "error": "Unauthorized",
  "message": "Token missing or expired."
}
```

**Technical Notes**

- Amounts must be in kobo (₦1000 = 100000).
- Do not expose your Paystack secret keys on the frontend.
- Guest and registered user workflow for payments is identical after the order is placed.
- For integration and testing Paystack, use your test keys and appropriate test cards.

**Need help?**  
Contact: api-support@mini-ecommerce.com for technical support or payment integration questions.

_Adapt field names and endpoint paths to match your implementation. This format is professional, clear, and guides both frontend and backend teams._
