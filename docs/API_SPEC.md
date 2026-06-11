# REST API Specification - BK Pergola et Alu

**Document Purpose:** Complete specification of all REST endpoints, request/response formats, status codes, and validation rules.

---

## 1. API Overview

### Base URL

**Development:**
```
http://localhost:8000/api
```

**Production:**
```
https://bk-pergola.tn/api
```

### Authentication

- **Type:** Session-based (PHP `$_SESSION`)
- **For admin endpoints:** Must have valid session (set after login)
- **Public endpoints:** No authentication required
- **Protected endpoints:** Require valid session, return 401 if not authenticated

### Response Format

**Success Response (200, 201):**
```json
{
  "success": true,
  "data": { /* resource data */ },
  "message": "Operation successful"
}
```

**Error Response (400, 401, 404, 500):**
```json
{
  "success": false,
  "error": "Error code",
  "message": "Human-readable error message",
  "details": { /* validation errors or extra info */ }
}
```

### Content-Type

- **Request:** `application/json` (or `multipart/form-data` for file uploads)
- **Response:** `application/json`

### Rate Limiting

- **Contact form:** 5 requests per IP per hour
- **Login attempts:** 5 attempts per IP per hour
- **Header:** `X-RateLimit-Remaining` in response

---

## 2. Authentication Endpoints

### POST /auth/login

**Purpose:** Admin login with username and password

**Access:** Public

**Request:**
```json
{
  "username": "adam",
  "password": "secure_password_here"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "username": "adam",
    "email": "adam@bk-pergola.tn"
  },
  "message": "Login successful"
}
```

**Response Headers:**
```
Set-Cookie: PHPSESSID=abc123...; HttpOnly; Secure; SameSite=Strict
```

**Error Responses:**

```json
// 400 - Invalid input
{
  "success": false,
  "error": "VALIDATION_ERROR",
  "message": "Username and password are required",
  "details": {
    "username": "This field is required",
    "password": "This field is required"
  }
}
```

```json
// 401 - Invalid credentials
{
  "success": false,
  "error": "INVALID_CREDENTIALS",
  "message": "Invalid username or password"
}
```

```json
// 429 - Rate limit exceeded
{
  "success": false,
  "error": "RATE_LIMIT",
  "message": "Too many login attempts. Try again later.",
  "retryAfter": 3600
}
```

**Notes:**
- Password must be bcrypt-verified against database
- Session created on successful login
- Session token stored in HTTP-only cookie (no JS access)
- Username case-insensitive

---

### POST /auth/logout

**Purpose:** Admin logout, destroy session

**Access:** Protected (requires valid session)

**Request:**
```
(empty body)
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Logout successful"
}
```

**Error Response (401):**
```json
{
  "success": false,
  "error": "UNAUTHORIZED",
  "message": "No active session"
}
```

**Notes:**
- Clears `$_SESSION`
- Clears session cookie

---

### GET /auth/me

**Purpose:** Get current authenticated admin (session validation)

**Access:** Protected

**Request:**
```
(no body)
```

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "username": "adam",
    "email": "adam@bk-pergola.tn",
    "isActive": true,
    "lastLogin": "2026-06-11T10:30:00Z"
  }
}
```

**Error Response (401):**
```json
{
  "success": false,
  "error": "UNAUTHORIZED",
  "message": "No active session or session expired"
}
```

---

## 3. Gallery Endpoints (Pinterest-Style)

### GET /gallery

**Purpose:** List all gallery items (with pagination, for Pinterest feed)

**Access:** Public

**Query Parameters:**
```
GET /api/gallery?page=1&perPage=30&sort=featured_desc
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | int | 1 | Page number for pagination |
| `perPage` | int | 30 | Items per page (max 100) |
| `sort` | string | featured_desc | Sort order: featured_desc, date_desc, date_asc |

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 1,
        "imagePath": "/uploads/gallery/2025/pergola-01.jpg",
        "thumbnailPath": "/uploads/thumbs/2025/pergola-01.jpg",
        "description": "Pergola moderne en aluminium blanc mat",
        "keywords": "pergola,aluminium,moderne",
        "featured": true,
        "projectId": null,
        "uploadedAt": "2026-06-10T14:30:00Z"
      },
      {
        "id": 2,
        "imagePath": "/uploads/gallery/2025/structure-01.jpg",
        "thumbnailPath": "/uploads/thumbs/2025/structure-01.jpg",
        "description": "Structure métallique pour terrasse",
        "keywords": "structure,métal,terrasse",
        "featured": false,
        "projectId": 1,
        "uploadedAt": "2026-06-09T10:15:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "perPage": 30,
      "total": 150,
      "totalPages": 5
    }
  }
}
```

**Notes:**
- Returns lightweight data (thumbnails, basic info)
- Featured items appear first
- Pagination for performance
- `projectId: null` = standalone photo

---

### GET /gallery/{id}

**Purpose:** Get single gallery item with full details and related project photos

**Access:** Public

**URL Parameters:**
```
GET /api/gallery/5
```

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "id": 5,
    "imagePath": "/uploads/gallery/2025/pergola-main.jpg",
    "thumbnailPath": "/uploads/thumbs/2025/pergola-main.jpg",
    "description": "Vue générale de la pergola installée. Dimensions: 6m x 4m.",
    "altText": "Pergola aluminium blanc mat vue de face",
    "keywords": "pergola,aluminium,blanc,vue-générale",
    "featured": true,
    "projectId": 2,
    "uploadedAt": "2026-06-08T09:45:00Z",
    "width": 1200,
    "height": 800,
    "project": {
      "id": 2,
      "title": "Pergola Moderne Tunis 2025",
      "description": "Magnifique pergola en aluminium blanc mat. Dimensions: 6m x 4m.",
      "category": "pergola",
      "location": "La Marsa, Tunis",
      "completionDate": "2025-02-20"
    },
    "relatedPhotos": [
      {
        "id": 6,
        "imagePath": "/uploads/gallery/2025/pergola-detail.jpg",
        "thumbnailPath": "/uploads/thumbs/2025/pergola-detail.jpg",
        "description": "Détail de la structure",
        "keywords": "pergola,structure,détail",
        "position": 1
      },
      {
        "id": 7,
        "imagePath": "/uploads/gallery/2025/pergola-evening.jpg",
        "thumbnailPath": "/uploads/thumbs/2025/pergola-evening.jpg",
        "description": "Pergola en soirée avec éclairage",
        "keywords": "pergola,éclairage",
        "position": 2
      }
    ]
  }
}
```

**Error Response (404):**
```json
{
  "success": false,
  "error": "NOT_FOUND",
  "message": "Gallery item not found"
}
```

**Notes:**
- Returns full image path (not thumbnail)
- If `projectId` is not null, returns related project photos
- Related photos ordered by `position` field
- Can be used for lightbox view

---

### GET /gallery/search

**Purpose:** Search gallery items by keywords or description

**Access:** Public

**Query Parameters:**
```
GET /api/gallery/search?q=pergola&page=1&perPage=30
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | Yes | Search query |
| `page` | int | No | Page number (default 1) |
| `perPage` | int | No | Items per page (default 30) |

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "query": "pergola",
    "items": [
      {
        "id": 1,
        "imagePath": "/uploads/gallery/2025/pergola-01.jpg",
        "thumbnailPath": "/uploads/thumbs/2025/pergola-01.jpg",
        "description": "Pergola moderne en aluminium blanc mat",
        "keywords": "pergola,aluminium,moderne",
        "featured": true,
        "uploadedAt": "2026-06-10T14:30:00Z"
      },
      {
        "id": 5,
        "imagePath": "/uploads/gallery/2025/pergola-main.jpg",
        "thumbnailPath": "/uploads/thumbs/2025/pergola-main.jpg",
        "description": "Vue générale de la pergola installée",
        "keywords": "pergola,aluminium,blanc,vue-générale",
        "featured": true,
        "uploadedAt": "2026-06-08T09:45:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "perPage": 30,
      "total": 12,
      "totalPages": 1
    }
  }
}
```

**Error Response (400):**
```json
{
  "success": false,
  "error": "VALIDATION_ERROR",
  "message": "Search query is required and must be at least 2 characters"
}
```

**Search Logic:**
- FULLTEXT search on `keywords` column (exact match preferred)
- OR search on `description` column (partial match)
- Case-insensitive
- Results sorted by relevance (keywords match first)

---

### POST /gallery (Admin Only)

**Purpose:** Upload new gallery item (standalone or add to project)

**Access:** Protected

**Request (multipart/form-data):**
```
POST /api/gallery

Headers:
  Content-Type: multipart/form-data
  Cookie: PHPSESSID=abc123...

Body:
  image: <file binary>
  description: "Pergola moderne avec éclairage intégré"
  keywords: "pergola,aluminium,éclairage,moderne"
  altText: "Pergola blanc mat vue de face"
  projectId: null (or integer ID to link to project)
  featured: false (or true)
```

**Form Fields:**

| Field | Type | Required | Validation |
|-------|------|----------|-----------|
| `image` | file | Yes | jpg, png, webp; <5MB; min 800px wide |
| `description` | string | No | Max 500 chars |
| `keywords` | string | No | Comma-separated tags; max 255 chars |
| `altText` | string | No | Max 255 chars (SEO) |
| `projectId` | integer | No | Must be valid project ID if provided |
| `featured` | boolean | No | Default false |

**Success Response (201):**
```json
{
  "success": true,
  "data": {
    "id": 150,
    "imagePath": "/uploads/gallery/2025/new-pergola-01.jpg",
    "thumbnailPath": "/uploads/thumbs/2025/new-pergola-01.jpg",
    "description": "Pergola moderne avec éclairage intégré",
    "keywords": "pergola,aluminium,éclairage,moderne",
    "altText": "Pergola blanc mat vue de face",
    "featured": false,
    "projectId": null,
    "uploadedAt": "2026-06-11T15:20:00Z"
  },
  "message": "Gallery item uploaded successfully"
}
```

**Error Responses:**

```json
// 400 - Validation error
{
  "success": false,
  "error": "VALIDATION_ERROR",
  "message": "Validation failed",
  "details": {
    "image": "File is required",
    "description": "Description must be less than 500 characters"
  }
}
```

```json
// 413 - File too large
{
  "success": false,
  "error": "FILE_TOO_LARGE",
  "message": "File exceeds 5MB limit"
}
```

```json
// 415 - Unsupported file type
{
  "success": false,
  "error": "UNSUPPORTED_FILE_TYPE",
  "message": "Only jpg, png, webp files are allowed"
}
```

```json
// 401 - Not authenticated
{
  "success": false,
  "error": "UNAUTHORIZED",
  "message": "Admin session required"
}
```

**Backend Processing:**
1. Validate file (type, size, dimensions)
2. Move to `/storage/gallery/`
3. Generate thumbnail (200x200px)
4. Optimize image (compression, WebP conversion)
5. Save metadata to `gallery_items` table
6. Return image paths

---

### PUT /gallery/{id} (Admin Only)

**Purpose:** Update gallery item details (description, keywords, featured flag)

**Access:** Protected

**Request:**
```json
PUT /api/gallery/5

{
  "description": "Updated description",
  "keywords": "pergola,aluminium,moderne,éclairage",
  "altText": "Updated alt text",
  "featured": true
}
```

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "id": 5,
    "description": "Updated description",
    "keywords": "pergola,aluminium,moderne,éclairage",
    "altText": "Updated alt text",
    "featured": true,
    "updatedAt": "2026-06-11T15:25:00Z"
  },
  "message": "Gallery item updated successfully"
}
```

**Error Responses:**

```json
// 404 - Item not found
{
  "success": false,
  "error": "NOT_FOUND",
  "message": "Gallery item not found"
}
```

```json
// 400 - Validation error
{
  "success": false,
  "error": "VALIDATION_ERROR",
  "message": "Validation failed",
  "details": {
    "keywords": "Keywords must be less than 255 characters"
  }
}
```

---

### DELETE /gallery/{id} (Admin Only)

**Purpose:** Delete gallery item (and image files)

**Access:** Protected

**Request:**
```
DELETE /api/gallery/5
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Gallery item deleted successfully"
}
```

**Error Response (404):**
```json
{
  "success": false,
  "error": "NOT_FOUND",
  "message": "Gallery item not found"
}
```

**Backend Processing:**
1. Find item by ID
2. Delete image files from disk
3. Delete thumbnail files
4. Delete record from `gallery_items` table
5. Confirm success

---

## 4. Project Endpoints

### GET /projects

**Purpose:** List all projects (with optional filtering)

**Access:** Public

**Query Parameters:**
```
GET /api/projects?category=pergola&featured=true&page=1
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `category` | string | Filter by category: pergola, structure, custom |
| `featured` | boolean | Filter featured only |
| `page` | int | Page number (default 1) |
| `perPage` | int | Items per page (default 20) |

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "projects": [
      {
        "id": 1,
        "title": "Pergola Tunis 2025",
        "description": "Magnifique pergola en aluminium blanc mat",
        "category": "pergola",
        "location": "La Marsa, Tunis",
        "completionDate": "2025-02-20",
        "featured": true,
        "imageCount": 5,
        "firstImage": "/uploads/thumbs/2025/pergola-main.jpg",
        "createdAt": "2026-06-08T09:45:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "perPage": 20,
      "total": 25,
      "totalPages": 2
    }
  }
}
```

---

### GET /projects/{id}

**Purpose:** Get single project with all associated gallery items

**Access:** Public

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "title": "Pergola Tunis 2025",
    "description": "Magnifique pergola en aluminium blanc mat. Dimensions: 6m x 4m. Finition: blanc mat.",
    "category": "pergola",
    "location": "La Marsa, Tunis",
    "completionDate": "2025-02-20",
    "featured": true,
    "createdAt": "2026-06-08T09:45:00Z",
    "galleryItems": [
      {
        "id": 5,
        "imagePath": "/uploads/gallery/2025/pergola-main.jpg",
        "thumbnailPath": "/uploads/thumbs/2025/pergola-main.jpg",
        "description": "Vue générale",
        "position": 0
      },
      {
        "id": 6,
        "imagePath": "/uploads/gallery/2025/pergola-detail.jpg",
        "thumbnailPath": "/uploads/thumbs/2025/pergola-detail.jpg",
        "description": "Détail de la structure",
        "position": 1
      }
    ]
  }
}
```

---

### POST /projects (Admin Only)

**Purpose:** Create new project

**Access:** Protected

**Request:**
```json
{
  "title": "New Project",
  "description": "Project description",
  "category": "pergola",
  "location": "Tunis",
  "completionDate": "2025-06-15",
  "featured": false
}
```

**Success Response (201):**
```json
{
  "success": true,
  "data": {
    "id": 50,
    "title": "New Project",
    "category": "pergola",
    "featured": false,
    "createdAt": "2026-06-11T15:30:00Z"
  },
  "message": "Project created successfully"
}
```

---

### PUT /projects/{id} (Admin Only)

**Purpose:** Update project details

**Access:** Protected

---

### DELETE /projects/{id} (Admin Only)

**Purpose:** Delete project (cascades to gallery items)

**Access:** Protected

**Notes:**
- Deleting project also deletes all associated gallery items
- Image files deleted from disk

---

## 5. Contact Endpoints

### POST /contacts

**Purpose:** Submit contact form (public lead capture)

**Access:** Public

**Rate Limit:** 5 requests per IP per hour

**Request:**
```json
{
  "name": "Ahmed Ben Ali",
  "email": "ahmed@example.tn",
  "phone": "+21622334455",
  "subject": "Demande de devis pergola",
  "message": "Bonjour, je suis intéressé par une pergola pour ma terrasse..."
}
```

**Validation Rules:**

| Field | Type | Required | Validation |
|-------|------|----------|-----------|
| `name` | string | Yes | Min 2, max 255 chars |
| `email` | string | Yes | Valid email format |
| `phone` | string | No | Valid phone format |
| `subject` | string | No | Max 255 chars |
| `message` | string | Yes | Min 10, max 5000 chars |

**Success Response (201):**
```json
{
  "success": true,
  "data": {
    "id": 245,
    "name": "Ahmed Ben Ali",
    "email": "ahmed@example.tn",
    "status": "new",
    "source": "form",
    "createdAt": "2026-06-11T16:00:00Z"
  },
  "message": "Message received. We will contact you soon!"
}
```

**Email Sent To:**
- `contact@bk-pergola.tn` with full contact details

**Error Response (400):**
```json
{
  "success": false,
  "error": "VALIDATION_ERROR",
  "message": "Validation failed",
  "details": {
    "name": "Name is required",
    "email": "Invalid email format",
    "message": "Message must be at least 10 characters"
  }
}
```

**Error Response (429 - Rate Limited):**
```json
{
  "success": false,
  "error": "RATE_LIMIT",
  "message": "Too many contact submissions. Try again later.",
  "retryAfter": 3600
}
```

**Backend Processing:**
1. Validate input
2. Sanitize HTML/scripts
3. Check IP rate limit
4. Save to `contacts` table (status: 'new')
5. Send email to admin
6. Return confirmation

---

### GET /contacts (Admin Only)

**Purpose:** List all contact submissions (lead dashboard)

**Access:** Protected

**Query Parameters:**
```
GET /api/contacts?status=new&source=form&page=1&perPage=50
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter: new, contacted, converted, spam |
| `source` | string | Filter: form, whatsapp, phone |
| `page` | int | Page number |
| `perPage` | int | Items per page (default 50) |

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "contacts": [
      {
        "id": 245,
        "name": "Ahmed Ben Ali",
        "email": "ahmed@example.tn",
        "phone": "+21622334455",
        "subject": "Demande de devis pergola",
        "message": "Bonjour, je suis intéressé...",
        "status": "new",
        "source": "form",
        "replied": false,
        "createdAt": "2026-06-11T16:00:00Z",
        "repliedAt": null
      }
    ],
    "pagination": {
      "page": 1,
      "perPage": 50,
      "total": 87,
      "totalPages": 2
    },
    "stats": {
      "totalCount": 87,
      "newCount": 15,
      "contactedCount": 42,
      "convertedCount": 28,
      "spamCount": 2,
      "bySource": {
        "form": 60,
        "whatsapp": 20,
        "phone": 7
      }
    }
  }
}
```

---

### PUT /contacts/{id} (Admin Only)

**Purpose:** Update contact status and reply flag

**Access:** Protected

**Request:**
```json
{
  "status": "contacted",
  "replied": true
}
```

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "id": 245,
    "status": "contacted",
    "replied": true,
    "repliedAt": "2026-06-11T16:30:00Z"
  },
  "message": "Contact updated successfully"
}
```

---

### GET /contacts/export (Admin Only)

**Purpose:** Export contacts as CSV (for CRM integration)

**Access:** Protected

**Query Parameters:**
```
GET /api/contacts/export?status=new&source=form&format=csv
```

**Success Response (200):**
```
Content-Type: text/csv
Content-Disposition: attachment; filename="contacts-2026-06-11.csv"

name,email,phone,subject,status,source,createdAt
Ahmed Ben Ali,ahmed@example.tn,+21622334455,Demande de devis pergola,new,form,2026-06-11T16:00:00Z
...
```

---

## 6. Team Endpoints

### GET /team

**Purpose:** List all team members

**Access:** Public

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "members": [
      {
        "id": 1,
        "name": "Mohamed Ben Khalifa",
        "title": "Directeur & Fondateur",
        "bio": "Expert en structures aluminium avec 15 ans d'expérience",
        "imagePath": "/uploads/team/mohamed.jpg",
        "position": 0,
        "email": "mohamed@bk-pergola.tn",
        "phone": "+21622334455"
      },
      {
        "id": 2,
        "name": "Fatima Dhouib",
        "title": "Ingénieur Technique",
        "bio": "Spécialiste en installation et maintenance",
        "imagePath": "/uploads/team/fatima.jpg",
        "position": 1,
        "email": "fatima@bk-pergola.tn",
        "phone": "+21699887766"
      }
    ]
  }
}
```

---

### POST /team (Admin Only)

**Purpose:** Add team member

**Access:** Protected

**Request (multipart/form-data):**
```
POST /api/team

Headers:
  Content-Type: multipart/form-data

Body:
  name: "New Team Member"
  title: "Job Title"
  bio: "Short biography"
  image: <file binary>
  email: "member@bk-pergola.tn"
  phone: "+216..."
  position: 2
```

---

### PUT /team/{id} (Admin Only)

**Purpose:** Update team member

**Access:** Protected

---

### DELETE /team/{id} (Admin Only)

**Purpose:** Delete team member

**Access:** Protected

---

## 7. Health & Status Endpoint

### GET /health

**Purpose:** Health check (uptime monitoring, load balancers)

**Access:** Public

**Success Response (200):**
```json
{
  "success": true,
  "status": "ok",
  "timestamp": "2026-06-11T16:45:00Z",
  "database": "connected",
  "cache": "ok"
}
```

**Error Response (503):**
```json
{
  "success": false,
  "status": "error",
  "message": "Database connection failed"
}
```

---

## 8. Error Handling

### Standard Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| VALIDATION_ERROR | 400 | Input validation failed |
| UNAUTHORIZED | 401 | Missing or invalid session |
| FORBIDDEN | 403 | Insufficient permissions |
| NOT_FOUND | 404 | Resource not found |
| CONFLICT | 409 | Resource already exists |
| RATE_LIMIT | 429 | Too many requests |
| FILE_TOO_LARGE | 413 | File exceeds size limit |
| UNSUPPORTED_FILE_TYPE | 415 | File type not allowed |
| INTERNAL_ERROR | 500 | Server error |

### Error Response Format

```json
{
  "success": false,
  "error": "ERROR_CODE",
  "message": "Human-readable message",
  "details": {
    "field1": "Error details",
    "field2": "Error details"
  },
  "timestamp": "2026-06-11T16:50:00Z"
}
```

---

## 9. Request/Response Examples

### Complete Flow: Search and View Gallery Item

**Step 1: Search for "pergola"**
```bash
GET /api/gallery/search?q=pergola&perPage=5 HTTP/1.1
Host: bk-pergola.tn
Accept: application/json
```

**Response:**
```json
{
  "success": true,
  "data": {
    "query": "pergola",
    "items": [
      {
        "id": 5,
        "imagePath": "/uploads/gallery/2025/pergola-main.jpg",
        "thumbnailPath": "/uploads/thumbs/2025/pergola-main.jpg",
        "description": "Pergola moderne",
        "keywords": "pergola,aluminium",
        "uploadedAt": "2026-06-08T09:45:00Z"
      }
    ],
    "pagination": { "page": 1, "perPage": 5, "total": 12, "totalPages": 3 }
  }
}
```

**Step 2: Click item to view full details**
```bash
GET /api/gallery/5 HTTP/1.1
Host: bk-pergola.tn
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 5,
    "imagePath": "/uploads/gallery/2025/pergola-main.jpg",
    "description": "Pergola moderne...",
    "keywords": "pergola,aluminium",
    "projectId": 2,
    "project": {
      "id": 2,
      "title": "Pergola Tunis 2025",
      "description": "...",
      "category": "pergola"
    },
    "relatedPhotos": [
      { "id": 6, "imagePath": "...", "position": 1 },
      { "id": 7, "imagePath": "...", "position": 2 }
    ]
  }
}
```

---

### Complete Flow: Admin Login and Upload Gallery Item

**Step 1: Login**
```bash
POST /api/auth/login HTTP/1.1
Host: bk-pergola.tn
Content-Type: application/json

{
  "username": "adam",
  "password": "secure_password"
}
```

**Response (sets session cookie):**
```
HTTP/1.1 200 OK
Set-Cookie: PHPSESSID=abc123...; HttpOnly; Secure; SameSite=Strict

{
  "success": true,
  "data": { "id": 1, "username": "adam" }
}
```

**Step 2: Upload gallery item**
```bash
POST /api/gallery HTTP/1.1
Host: bk-pergola.tn
Content-Type: multipart/form-data
Cookie: PHPSESSID=abc123...

--boundary
Content-Disposition: form-data; name="image"; filename="pergola.jpg"
Content-Type: image/jpeg

[binary file data]
--boundary
Content-Disposition: form-data; name="description"

Pergola modern avec éclairage
--boundary
Content-Disposition: form-data; name="keywords"

pergola,aluminium,éclairage
--boundary
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 150,
    "imagePath": "/uploads/gallery/2025/pergola.jpg",
    "thumbnailPath": "/uploads/thumbs/2025/pergola.jpg"
  },
  "message": "Gallery item uploaded successfully"
}
```

**Step 3: Logout**
```bash
POST /api/auth/logout HTTP/1.1
Host: bk-pergola.tn
Cookie: PHPSESSID=abc123...
```

---

## 10. CORS & Security Headers

### CORS (Cross-Origin)

**Development (same domain):**
- No special CORS headers needed
- Frontend and backend on same origin

**Future Mobile App:**
```
Access-Control-Allow-Origin: https://app.bk-pergola.tn
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, X-CSRF-Token
Access-Control-Allow-Credentials: true
```

### Security Headers

**All Responses:**
```
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Content-Security-Policy: default-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

---

## 11. Pagination

### Pagination Parameters

```
GET /api/gallery?page=2&perPage=30
```

### Pagination Response

```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "perPage": 30,
    "total": 150,
    "totalPages": 5,
    "hasNextPage": true,
    "hasPrevPage": true,
    "nextPageUrl": "/api/gallery?page=3&perPage=30",
    "prevPageUrl": "/api/gallery?page=1&perPage=30"
  }
}
```

**Rules:**
- `page` defaults to 1
- `perPage` defaults to 30 (max 100)
- `total` = total records matching query
- `totalPages` = ceil(total / perPage)

---

## 12. Versioning & Changelog

**API Version:** 1.0

**Stability:** Production

**Changelog:**
- v1.0 (June 2026): Initial release with gallery, contacts, projects, team endpoints

---

**API Specification Version:** 1.0  
**Last Updated:** June 2026  
**Status:** Production-ready
