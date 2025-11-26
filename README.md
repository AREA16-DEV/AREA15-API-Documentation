Serverless API for AREA15 ticketing and checkout operations built with AWS Lambda and API Gateway using the Serverless Framework.

## Table of Contents

- [Authentication](#authentication)
- [API Endpoints](#api-endpoints)
  - [Stripe & Hardware](#stripe--hardware)
  - [Events & Sessions](#events--sessions)
  - [Cart Operations](#cart-operations)
  - [User Management](#user-management)
  - [Gift Cards](#gift-cards)
  - [Payment Processing](#payment-processing)
  - [Content & Kiosk](#content--kiosk)
  - [HubSpot Integration](#hubspot-integration)
  - [UDX Integration](#udx-integration)
  - [Upsell](#upsell)

## Authentication

All endpoints require an API key to be included in the request headers. Include the API key using the `x-api-key` header:

```
x-api-key: <your-api-key>
```

## API Endpoints

### Stripe & Hardware

#### `POST /connect`

Get Stripe hardware connection token for payment terminal integration.

**Request Body:** None (empty object)

**Response:**
```json
{
  "secret": "<stripe-connection-token>"
}
```

---

### Events & Sessions

#### `POST /events`

Retrieve available events. Optionally filter by date for today's events.

**Request Body:**
```json
{
  "date": "2024-01-15",  // Optional: ISO date string
  "todaysEvents": true   // Optional: boolean to filter today's events
}
```

**Response:** Array of available events with ticket groups, ticket types, and metadata.

---

#### `POST /sessions`

Get available sessions for a specific event on a given date.

**Request Body:**
```json
{
  "date": "2024-01-15",  // Required: ISO date string
  "id": "event-id"       // Required: Event ID
}
```

**Response:** Array of available sessions for the event.

---

#### `POST /types`

Retrieve ticket types for a specific session.

**Request Body:**
```json
{
  "session": "session-id"  // Required: Session ID
}
```

**Response:** Session details with ticket groups and ticket types.

---

#### `POST /message-rules`

Get message rules for a specific event.

**Request Body:**
```json
{
  "event_id": "event-id"  // Required: Event ID
}
```

**Response:** Event details with message rules.

---

### Cart Operations

#### `POST /addtocart`

Add tickets to a cart. Creates a new cart if `id` is not provided.

**Request Body:**
```json
{
  "id": "cart-id",              // Optional: Existing cart ID (creates new if omitted)
  "session_id": "session-id",   // Required: Session ID
  "tickets": [                  // Required: Array of ticket objects
    {
      "ticket_type_id": "type-id",
      "quantity": 2
    }
  ]
}
```

**Response:** Updated cart with reserved tickets.

---

#### `POST /remove-from-cart`

Remove tickets from a cart.

**Request Body:**
```json
{
  "id": "cart-id",     // Required: Cart ID
  "tickets": "[...]"   // Required: JSON stringified array of ticket IDs to remove
}
```

**Response:** Updated cart with tickets removed.

---

#### `POST /promo-code`

Apply a promotional code to a cart.

**Request Body:**
```json
{
  "id": "cart-id",    // Required: Cart ID
  "code": "PROMO123"  // Required: Promotional code
}
```

**Response:** Updated cart with promo code applied.

---

### User Management

#### `POST /upsert-user`

Create or update a user in the ticketing system. **The cart will automatically be assigned to the user after creation/update.**

**Request Body:**
```json
{
  "customer": {
    "first_name": "John",      // Required
    "last_name": "Doe",        // Required
    "email": "john@example.com", // Required
    "phone": "555-1234",       // Required
    "postal": "89101",         // Required: ZIP code
    "birthdate": "1990-01-01", // Required: Date string
    "optin": true              // Required: Boolean for marketing opt-in
  },
  "cartId": "cart-id"          // Required: Cart ID to assign to the user
}
```

**Response:** User creation/update confirmation. The cart is automatically assigned to the user.

---

#### `POST /customerlookup`

Look up a customer by email address.

**Request Body:**
```json
{
  "email": "customer@example.com"  // Required: Email address
}
```

**Response:** Customer identity information if found.

---

### Gift Cards

#### `POST /gift-card`

Validate and retrieve gift card information.

**Request Body:**
```json
{
  "code": "gift-card-code"  // Required: Gift card code
}
```

**Response:** Gift card details including balance and status.

---

#### `POST /gift-card-pay`

Apply gift card payment to a cart (full or partial payment).

**Request Body:**
```json
{
  "code": {              // Required: Gift card object from /gift-card response
    "id": "card-id",
    "value": 50.00
  },
  "cartId": "cart-id",   // Required: Cart ID
  "partial": false        // Optional: Boolean for partial payment
}
```

**Response:** Payment confirmation and updated cart.

---

### Payment Processing

#### `POST /intent`

Create a Stripe payment intent for card-present transactions.

**Request Body:**
```json
{
  "cart_id": "cart-id"  // Required: Cart ID
}
```

**Response:** Stripe payment intent details for card reader integration.

---

#### `POST /checkoutcart`

Complete cart checkout with payment.

**Request Body:**
```json
{
  "cart_id": "cart-id",              // Required: Cart ID
  "payment_intent_id": "pi_xxx"      // Optional: Stripe payment intent ID (if not provided, processes as free)
}
```

**Response:** Order confirmation with ticket order details.

---

### Content & Kiosk

#### `POST /terms-and-privacy`

Fetch terms and conditions or privacy policy content from AREA15 website.

**Request Body:**
```json
{
  "type": "terms"  // Required: Either "terms" or "privacy"
}
```

**Response:** HTML content string (sanitized, links removed).

---

#### `POST /kiosk-details`

Get kiosk configuration and details by kiosk ID.

**Request Body:**
```json
{
  "kiosk_id": "kiosk-unit-id"  // Required: Kiosk unit identifier
}
```

**Response:** Kiosk configuration including ad campaigns, banners, reader settings, and vendor information.

---

#### `GET /get-cached`

Retrieve cached events from Redis cache.

**Request Body:** None

**Response:** Cached events data or error if cache is empty.

---

### HubSpot Integration

#### `POST /hubspot-check`

Check email communication preferences in HubSpot.

**Request Body:**
```json
{
  "email": "user@example.com"  // Required: Email address
}
```

**Response:** HubSpot communication preference status.

---

#### `POST /user-profile`

Create or update a user profile in HubSpot.

**Request Body:**
```json
{
  "email": "user@example.com",  // Required: Email address
  "properties": [                // Required: Array of HubSpot contact properties
    {
      "property": "firstname",
      "value": "John"
    },
    {
      "property": "lastname",
      "value": "Doe"
    }
  ]
}
```

**Response:** HubSpot contact creation/update confirmation.

---

#### `POST /optin`

Subscribe an email to HubSpot marketing communications.

**Request Body:**
```json
{
  "email": "user@example.com"  // Required: Email address
}
```

**Response:** Subscription confirmation.

---

### UDX Integration

#### `POST /udxauth`

Authenticate with UDX (Universal Studios Experience) API and get access token.

**Request Body:** None

**Response:** UDX authentication token.

---

#### `POST /udxevents`

Retrieve UDX ticket products using authentication token.

**Request Body:**
```json
{
  "token": "udx-auth-token"  // Required: Token from /udxauth endpoint
}
```

**Response:** UDX ticket products data.

---

### Upsell

#### `POST /upsell`

Get upsell recommendations for a cart.

**Request Body:**
```json
{
  "cartId": "cart-id"  // Required: Cart ID
}
```

**Response:** Upsell product recommendations.

---

## Error Responses

All endpoints may return the following error responses:

- **401 Unauthorized:** Missing or invalid API key
- **400 Bad Request:** Invalid or missing required parameters
- **500 Internal Server Error:** Server error with error message in response body

Error response format:
```json
{
  "error": "Error message description"
}
```

## CORS

All endpoints support CORS and include appropriate headers for cross-origin requests. OPTIONS preflight requests are automatically handled.

---

## WEBAPP NOTES

Our webapps utilize **@stripe/react-stripe-js** 

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISH_KEY, {
    stripeAccount: process.env.NEXT_PUBLIC_STRIPE_ACCOUNT
});

## KIOSK APP NOTES

Our Physical reader applicaitons utlize **@capacitor-community/stripe-terminal** for Android and **@stripe/terminal-js** for Web-based 

---

## REQUIRED TICKET FLOW

1. Get Events, Sessions, Ticket Groups, Ticket Types
2. Add/Remove Tickets to cart based on selected session and Ticket Types (This process will automatically create a cart)
3. Ingest user info (First Name, Last Name, Email, Phone (optional), Zip (optional), Birthdate (optional)) (This process will create a user in our system and attach them to the cart)
4. Allow for Promo code or Giftcard inputs (Promo code can be added to cart any time before checkout and will return adjusted cart values/fees -- Giftcards will need to be tracked on frontend until checkout) 
5. Start of checkout, if there are giftcards, check value of giftcard and use appropriate chckout parameters based on full payment or partial payment
6. Final Checkout with any left over after promos and giftcards
7. User will get automatic emails from AREA15 on successful checkout


