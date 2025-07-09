# MealPe <> POS Integration Documentation

## 1. Purpose and Scope
This document provides technical guidance for POS (Point of Sale) system providers to integrate with MealPe. It details the data exchanged for menu, order, and outlet events, including payload structures, authentication, and webhook requirements.

---

## 2. Integration Architecture

The integration is bi-directional:
- **MealPe pushes data** (menu, order, outlet status) to the POS via API endpoints.
- **POS systems send data** (menu updates, order status, item/outlet status) to MealPe via webhooks.

```mermaid
graph TD
  A[MealPe] -- Pushes Menu/Order/Outlet Data --> B(POS API)
  B -- Sends Webhook Events (Order Status, Menu Updates, Outlet Status) --> A
```

---

## 3.  
- The access token must be included in the HTTP header:
  - `Authorization: Bearer <access_token>`
- Access tokens are securely issued to each integration partner (POS or outlet) by MealPe during onboarding.
- Both MealPe and POS systems must validate the access token on every incoming request before processing.
- Tokens should be kept confidential and rotated periodically for security.
- If the access token is missing or invalid, the request must be rejected with an appropriate error response.

**Example header:**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## 4. Webhook Requirements
- POS systems **must** implement webhook endpoints to receive real-time updates from MealPe.
- Webhook endpoints must be accessible over HTTPS.
- Webhook payloads are sent as JSON.
- Respond with `{ success: true, message: "..." }` on successful processing.

---

## 5. Event Data Exchange

### 5.1 Menu Events

#### 5.1.1 Data We Send to POS (Menu Push)
Endpoint: `/pushData`

**Example Payload:**
```json
{
  "success": 1,
  "restaurant": {
    "outletId": "<outletId>",
    "active": true,
    "details": {
      "restaurantName": "Sample Restaurant",
      "address": "123 Main St",
      "contact": "9876543210",
      "currency_html": "₹",
      "country": "India",
      ...
    }
  },
  "categories": [
    {
      "categoryId": "101",
      "active": true,
      "categoryName": "Starters",
      ...
    }
  ],
  "items": [
    {
      "itemId": "501",
      "itemName": "Paneer Tikka",
      "price": 250,
      "active": true,
      "categoryId": "101",
      ...
    }
  ],
  ...
}
```

**Field Explanations:**
- `restaurant`: Restaurant/outlet details.
- `categories`: Menu categories.
- `items`: Menu items with pricing, status, and category linkage.
- Additional fields: `variations`, `addOnGroups`, `attributes`, `taxes`, etc.

#### 5.1.2 Data We Expect from POS (Menu Webhook)
Webhook: `/menu-sharing-webhook`

**Example Payload:**
```json
{
  "parentCategories": [ ... ],
  "categories": [ ... ],
  "items": [ ... ],
  "addOnGroups": [ ... ]
}
```
- Each array contains objects with fields like `id`, `name`, `status`, `categoryId`, `itemId`, etc.
- POS should send updated menu data in this format when menu changes occur.

---

### 5.2 Order Events

#### 5.2.1 Data We Send to POS (Order Push)
Endpoint: `/save_order`

**Example Payload:**
```json
{
  "orderInfo": {
    "restaurant": { "details": { ... } },
    "customer": { "details": { ... } },
    "order": { "details": { ... } },
    "orderItems": [ ... ],
    "taxes": [ ... ]
  },
  "udid": "",
  "deviceType": "Web"
}
```

**Order Details Example:**
```json
{
  "orderId": "ORD12345",
  "createdOn": "2024-06-01 12:34:56",
  "orderType": "Delivery",
  "paymentType": "ONLINE",
  "total": "500.00",
  "taxTotal": "25.00",
  "packingCharges": "10.00",
  "deliveryCharges": "40.00",
  "description": "No onions",
  "callbackUrl": "https://yourdomain.com/webhook/order-status/ORD12345",
  ...
}
```

**OrderItem Example:**
```json
{
  "itemId": "501",
  "itemName": "Paneer Tikka",
  "price": 250,
  "quantity": 2,
  "finalPrice": 500,
  ...
}
```

#### 5.2.2 Data We Expect from POS (Order Status Webhook)
Webhook: `/order-status-webhook`

**Example Payload:**
```json
{
  "orderId": "ORD12345",
  "status": "accepted", // or "preparing", "ready", "completed", "cancelled"
  "cancelReason": "",
  "minimumPrepTime": 20,
  "riderName": "John Doe",
  "riderPhoneNumber": "9876543210",
  ...
}
```
- POS should notify us of order status changes using this format.

---

### 5.3 Outlet/Store Events

#### 5.3.1 Data We Send to POS (Outlet Status Push)
Endpoint: `/get-store-status-webhook`

**Example Payload:**
```json
{
  "outletId": "<outletId>"
}
```
**Response Expected:**
```json
{
  "http_code": 200,
  "status": "success",
  "storeStatus": "1", // 1=open, 0=closed
  "message": "Store Delivery Status fetched successfully."
}
```

#### 5.3.2 Data We Expect from POS (Outlet Status Update Webhook)
Webhook: `/update-store-status-webhook`

**Example Payload:**
```json
{
  "outletId": "<outletId>",
  "storeStatus": 1, // 1=open, 0=closed
  "reason": "Maintenance",
  "turnOnTime": "2024-06-01 18:00:00"
}
```
- POS should notify us when outlet status changes (open/close) with reason and optional turn-on time.

---

## 6. Data Structure Reference

### 6.1 Menu Data Fields
- `outletId`, `active`, `details`: Outlet info
- `categories`: `categoryId`, `categoryName`, `active`, etc.
- `items`: `itemId`, `itemName`, `price`, `active`, `categoryId`, etc.
- `variations`, `addOnGroups`, `attributes`, `taxes`: As per menu complexity

### 6.2 Order Data Fields
- `orderId`, `createdOn`, `orderType`, `paymentType`, `total`, `taxTotal`, `packingCharges`, `deliveryCharges`, `description`, `callbackUrl`, etc.
- `orderItems`: Array of items with `itemId`, `itemName`, `price`, `quantity`, etc.
- `taxes`: Array of tax objects with `id`, `title`, `type`, `price`, `tax`, etc.

### 6.3 Outlet/Store Data Fields
- `outletId`: Unique outlet/restaurant identifier
- `storeStatus`: 1=open, 0=closed
- `reason`: Reason for status change
- `turnOnTime`: Scheduled reopen time (if applicable)

---

## 7. Error Handling
- All endpoints should return a JSON object with at least:
  - `success`: boolean
  - `message`: string
- On error, include an `error` field with details.
- Example:
```json
{
  "success": false,
  "message": "Invalid request",
  "error": "Missing required field"
}
```

---

## 8. Route Implementation Patterns
- All endpoints must validate the access token in the `Authorization` header before processing requests.
- All responses must include `success` and `message` fields.
- Use appropriate HTTP status codes (200 for success, 4xx/5xx for errors).
- Validate and sanitize all input data.
- Use try-catch blocks for error handling and log errors for debugging.

---

## 9. Additional Notes
- Webhook URLs are provisioned per integration. 
