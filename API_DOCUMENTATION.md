# API Documentation - Landing Page Payment & Related APIs

This document provides cURL examples for all APIs related to the landing page payment system, voucher verification, and course variant stock management.

## Base URL
All API endpoints are relative to your base URL. Replace `https://api.persianball.ir` with your actual domain.

---

## 1. Landing Page Order API

Create a guest order for a course variant from a landing page.

### Endpoint
```
POST /api/order/landing-page-order/
```

### Authentication
None required (public endpoint)

### Request Body
```json
{
  "course_variant_id": 123,
  "first_name": "علی",
  "last_name": "احمدی",
  "email": "ali@example.com",
  "mobile_number": "09123456789",
  "voucher_code": "DISCOUNT10"
}
```

**Fields:**
- `course_variant_id` (integer, required): ID of the CourseVariant to purchase
- `first_name` (string, required, max 50): Customer's first name
- `last_name` (string, required, max 50): Customer's last name
- `email` (string, optional): Customer's email address
- `mobile_number` (string, optional, max 15): Customer's mobile number
- `voucher_code` (string, optional): Voucher/discount code to apply

**Note:** At least one of `email` or `mobile_number` must be provided.

### cURL Example
```bash
curl -X POST https://api.persianball.ir/v1/order/landing-page-order/ \
  -H "Content-Type: application/json" \
  -d '{
    "course_variant_id": 123,
    "first_name": "علی",
    "last_name": "احمدی",
    "email": "ali@example.com",
    "mobile_number": "09123456789",
    "voucher_code": "DISCOUNT10"
  }'
```

### Success Response (201 Created)
```json
{
  "redirect_url": "https://api.persianball.ir/payment/to_bank/<payment-uuid>"
}
```

For free courses (price = 0):
```json
{
  "status": "done"
}
```

### Error Responses

**Missing Contact Info (400 Bad Request):**
```json
{
  "detail": "Either email or mobile_number must be provided",
  "code": "missing_contact_info"
}
```

**Course Variant Not Found (400 Bad Request):**
```json
{
  "detail": "Course variant does not exist",
  "code": "course_variant_not_found"
}
```

**Course Variant Inactive (400 Bad Request):**
```json
{
  "detail": "Course variant is not active",
  "code": "course_variant_inactive"
}
```

**Course Sold Out (400 Bad Request):**
```json
{
  "detail": "Course variant is sold out. No available spots.",
  "code": "course_variant_sold_out"
}
```

**Invalid Voucher (400 Bad Request):**
```json
{
  "detail": "Discount does not exist",
  "code": "invalid"
}
```

---

## 2. Voucher Verification API

Verify a voucher code and get calculated prices with discount for a specific course variant.

### Endpoint
```
POST /api/payment/voucher/verify/
```

### Authentication
None required (public endpoint)

### Request Body
```json
{
  "code": "DISCOUNT10",
  "course_variant_id": 123
}
```

**Fields:**
- `code` (string, required): Voucher code to verify
- `course_variant_id` (integer, required): ID of the CourseVariant to calculate discount for

### cURL Example
```bash
curl -X POST https://api.persianball.ir/v1/payment/voucher/verify/ \
  -H "Content-Type: application/json" \
  -d '{
    "code": "DISCOUNT10",
    "course_variant_id": 123
  }'
```

### Success Response (200 OK)
```json
{
  "discount_percent": 10,
  "original_price": 100000,
  "discount_amount": 9000,
  "final_price": 91000
}
```

**Response Fields:**
- `discount_percent`: Voucher discount percentage
- `original_price`: Original course variant price (after course discount if any)
- `discount_amount`: Discount amount in Toman
- `final_price`: Final price after voucher discount

### Error Responses

**Voucher Not Found (400 Bad Request):**
```json
{
  "detail": "Discount does not exist",
  "code": "invalid"
}
```

**Voucher Not Active (400 Bad Request):**
```json
{
  "detail": "Discount is not active",
  "code": "inactive_voucher"
}
```

**Voucher Expired (400 Bad Request):**
```json
{
  "detail": "Discount expired",
  "code": "discount_expired"
}
```

**Voucher Capacity Full (400 Bad Request):**
```json
{
  "detail": "Discount capacity is full",
  "code": "discount_full_capacity"
}
```

**Course Variant Not Found (400 Bad Request):**
```json
{
  "detail": "Course variant does not exist",
  "code": "course_variant_not_found"
}
```

**Course Variant Inactive (400 Bad Request):**
```json
{
  "detail": "Course variant is not active",
  "code": "course_variant_inactive"
}
```

---

## 3. Course Variant Stock Information API

Get stock information for a course variant including total capacity, purchased count, and available spots.

### Endpoint
```
GET /api/product/course-variant/{id}/stock-info/
```

### Authentication
None required (public endpoint)

### URL Parameters
- `id` (integer, required): CourseVariant ID

### cURL Example
```bash
curl -X GET https://api.persianball.ir/v1/product/course-variant/123/stock-info/ \
  -H "Content-Type: application/json"
```

### Success Response (200 OK)
```json
{
  "total_capacity": 100,
  "purchased_count": 45,
  "available_spots": 55
}
```

**Response Fields:**
- `total_capacity`: Total number of available spots for this course variant
- `purchased_count`: Number of spots purchased via landing page orders (completed orders only)
- `available_spots`: Remaining available spots (total_capacity - purchased_count)

### Error Response

**Course Variant Not Found (404 Not Found):**
```json
{
  "detail": "Course variant not found"
}
```

---

## Complete Flow Example

### Step 1: Check Stock Availability
```bash
curl -X GET https://api.persianball.ir/v1/product/course-variant/123/stock-info/
```

Response:
```json
{
  "total_capacity": 100,
  "purchased_count": 45,
  "available_spots": 55
}
```

### Step 2: Verify Voucher (Optional)
```bash
curl -X POST https://api.persianball.ir/v1/payment/voucher/verify/ \
  -H "Content-Type: application/json" \
  -d '{
    "code": "DISCOUNT10",
    "course_variant_id": 123
  }'
```

Response:
```json
{
  "discount_percent": 10,
  "original_price": 100000,
  "discount_amount": 10000,
  "final_price": 90000
}
```

### Step 3: Create Landing Page Order
```bash
curl -X POST https://api.persianball.ir/v1/order/landing-page-order/ \
  -H "Content-Type: application/json" \
  -d '{
    "course_variant_id": 123,
    "first_name": "علی",
    "last_name": "احمدی",
    "email": "ali@example.com",
    "mobile_number": "09123456789",
    "voucher_code": "DISCOUNT10"
  }'
```

Response:
```json
{
  "redirect_url": "https://api.persianball.ir/payment/to_bank/abc123def456"
}
```

### Step 4: User Redirects to Payment Gateway
The user is redirected to the `redirect_url` to complete payment.

### Step 5: Payment Callback
After successful payment, the system:
1. Verifies payment with gateway
2. Finds or creates user account (by email/mobile)
3. Assigns course to user
4. Decreases course variant capacity
5. Sends SMS with credentials (if new user and mobile provided)

---

## Notes

1. **Landing Page Orders Only**: Only orders created via the landing page API decrease the course variant capacity. Regular authenticated orders do not affect capacity.

2. **User Creation**: If a user doesn't exist, a new account is automatically created with:
   - Random username (UUID)
   - Random password (10 characters, alphanumeric)
   - SMS sent with credentials (if mobile number provided)

3. **Voucher Application**: Vouchers are applied after course variant discounts. The discount is calculated on the price after the course's own discount.

4. **Stock Tracking**: `purchased_count` only includes completed orders from landing page API. Orders with status 'done' are counted.

5. **Capacity Validation**: The landing page API validates available capacity before creating an order. If `available_spots <= 0`, the order creation is rejected.

