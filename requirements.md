## 🔐 1. User Authentication

### 🔧 API Endpoints:

* `POST /api/auth/register`
* `POST /api/auth/login`
* `POST /api/auth/oauth/google`
* `GET /api/auth/profile`

### 📥 Input Specifications:

* `email` (string, required, valid email)
* `password` (string, required, min 8 chars)
* `role` (enum: `guest` | `host`)
* `name`, `profile_photo_url` (optional)

### 📤 Output Specifications:

* `200 OK`: `{ token: JWT, user: { id, name, email, role } }`
* `401 Unauthorized`: invalid credentials
* `400 Bad Request`: validation errors

### ✅ Validation Rules:

* Unique email constraint
* Secure password (hashing with bcrypt)
* Role must be either `guest` or `host`

### 🚀 Performance Criteria:

* Token generation under 200ms
* Auth endpoints should support 100+ concurrent sessions
* Use Redis cache for token/session verification (optional)

---

## 🏠 2. Property Management

### 🔧 API Endpoints:

* `POST /api/properties`
* `PUT /api/properties/:id`
* `DELETE /api/properties/:id`
* `GET /api/properties?filters`

### 📥 Input Specifications:

* `title`, `description`, `location`, `price`, `amenities[]`, `availability[]`
* `images[]` (URLs or file upload references)
* `host_id` (derived from JWT)

### 📤 Output Specifications:

* `201 Created`: full property object
* `404 Not Found`: if property does not exist
* `403 Forbidden`: if host does not own the property

### ✅ Validation Rules:

* Title required, 100 char max
* Price: positive float
* Availability: date range validation (no overlaps)

### 🚀 Performance Criteria:

* Search with filters should return results within 500ms
* Listing changes indexed within 10s (for search cache)

---

## 📅 3. Booking System

### 🔧 API Endpoints:

* `POST /api/bookings`
* `GET /api/bookings/:id`
* `PUT /api/bookings/:id/cancel`
* `GET /api/bookings?userId={}`

### 📥 Input Specifications:

* `property_id`, `start_date`, `end_date`
* `guest_id` (derived from JWT)

### 📤 Output Specifications:

* `201 Created`: booking object with status `pending`
* `409 Conflict`: date overlaps
* `200 OK`: status update or cancellation

### ✅ Validation Rules:

* Dates must be in future, and not overlap with confirmed bookings
* Guest cannot book own listing
* Booking status updated via payment webhook (confirmed/cancelled)

### 🚀 Performance Criteria:

* Booking transaction latency < 300ms (excluding payment)
* Double-booking conflict detection must use DB locks or atomic checks

