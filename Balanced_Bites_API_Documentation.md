# The Balanced Bites API Documentation

## 1. Overview

The Balanced Bites backend is a RESTful API built with **Node.js**, **Express**, and **MongoDB/Mongoose**.  
It supports authentication, recipe management, personalised meal planning, grocery list generation, and admin features.

### Main capabilities
- User registration and login with JWT authentication
- User profile onboarding and updates
- Public recipe browsing
- Admin recipe CRUD operations
- Spoonacular recipe search integration
- Personal meal plan saving and retrieval
- Grocery list checkbox updates
- Admin dashboard and user management

---

## 2. Base URLs

### Backend development URL
`http://localhost:5000`

### API base path
`http://localhost:5000/api/v1`

### Health check
`GET /api/health`

Example:
```http
GET http://localhost:5000/api/health
```

Success response:
```json
{
  "success": true,
  "message": "🌿 Balanced Bites API is running!"
}
```

---

## 3. Authentication

The API uses **JWT bearer tokens** for protected routes.

### Header format
```http
Authorization: Bearer <accessToken>
```

### Token flow
- `POST /auth/register` returns an **access token** and a **refresh token**
- `POST /auth/login` returns an **access token** and a **refresh token**
- `POST /auth/refresh` exchanges a valid refresh token for a new token pair
- `POST /auth/logout` clears the saved refresh token for the current user

### Access control
- **Public routes**: recipe browsing, recipe by ID, health check
- **Authenticated routes**: profile, Spoonacular search, meal plan operations
- **Admin-only routes**: recipe create/update/delete and all `/admin` endpoints

---

## 4. Security Features

The backend includes:
- **Helmet** for secure HTTP headers
- **CORS** allowlist configuration
- **Rate limiting** on `/api/*`
- Stronger auth rate limiting on:
  - `/api/v1/auth/login`
  - `/api/v1/auth/register`
- **bcrypt** password hashing
- JWT-based authentication middleware
- Role-based authorisation for admin-only routes

---

## 5. Standard Response Pattern

Most endpoints return JSON in a similar format:

### Success
```json
{
  "success": true,
  "data": {}
}
```

### Error
```json
{
  "success": false,
  "message": "Description of the error"
}
```

Note: some endpoints return resource-specific keys such as `user`, `recipe`, `recipes`, `plan`, `stats`, or `groceryList` instead of a generic `data` field.

---

## 6. Data Models

## 6.1 User
```json
{
  "_id": "661111111111111111111111",
  "name": "Alex",
  "email": "alex@example.com",
  "role": "user",
  "profile": {
    "gender": "male",
    "age": 19,
    "weight": 75,
    "weightUnit": "kg",
    "height": 188,
    "heightUnit": "cm",
    "goal": "bulk",
    "workoutFreq": "4-5 times/week",
    "workoutDuration": "60 mins",
    "diet": ["vegetarian"],
    "allergies": ["nuts"]
  },
  "isOnboarded": true,
  "createdAt": "2026-04-06T10:00:00.000Z",
  "updatedAt": "2026-04-06T10:10:00.000Z"
}
```

## 6.2 Recipe
```json
{
  "_id": "662222222222222222222222",
  "name": "Chicken Rice Bowl",
  "description": "High-protein balanced meal",
  "emoji": "🍗",
  "tag": "High Protein",
  "calories": 540,
  "prepTime": "25 min",
  "servings": 2,
  "goal": "bulk",
  "diet": ["gluten-free"],
  "allergies": [],
  "ingredients": [
    { "name": "Chicken breast", "amount": "200g" },
    { "name": "Rice", "amount": "150g" }
  ],
  "steps": [
    "Cook the rice",
    "Grill the chicken",
    "Assemble the bowl"
  ],
  "nutrition": {
    "protein": 42,
    "carbs": 48,
    "fat": 12,
    "fibre": 4
  },
  "sourceId": "12345",
  "color": "#E8F5E9",
  "createdBy": "661111111111111111111111",
  "createdAt": "2026-04-06T10:00:00.000Z",
  "updatedAt": "2026-04-06T10:10:00.000Z"
}
```

## 6.3 Meal Plan
```json
{
  "_id": "663333333333333333333333",
  "userId": "661111111111111111111111",
  "weekStart": "2026-04-06T00:00:00.000Z",
  "slots": [
    {
      "day": "Mon",
      "meal": "Breakfast",
      "recipe": "662222222222222222222222"
    },
    {
      "day": "Mon",
      "meal": "Lunch",
      "recipe": "662222222222222222222223"
    }
  ],
  "groceryList": [
    {
      "name": "Chicken breast",
      "amount": "200g",
      "category": "General",
      "checked": false
    }
  ],
  "createdAt": "2026-04-06T10:00:00.000Z",
  "updatedAt": "2026-04-06T10:10:00.000Z"
}
```

---

## 7. Endpoint Summary

| Method | Endpoint | Access | Description |
|---|---|---:|---|
| GET | `/api/health` | Public | Health check |
| POST | `/api/v1/auth/register` | Public | Register a new user |
| POST | `/api/v1/auth/login` | Public | Log in and receive tokens |
| POST | `/api/v1/auth/refresh` | Public | Refresh access token |
| GET | `/api/v1/auth/profile` | Authenticated | Get current user profile |
| PUT | `/api/v1/auth/profile` | Authenticated | Update current user profile |
| POST | `/api/v1/auth/logout` | Authenticated | Log out current user |
| GET | `/api/v1/recipes` | Public | List recipes with optional filters |
| GET | `/api/v1/recipes/:id` | Public | Get one recipe by ID |
| POST | `/api/v1/recipes` | Admin | Create a recipe |
| PUT | `/api/v1/recipes/:id` | Admin | Update a recipe |
| DELETE | `/api/v1/recipes/:id` | Admin | Delete a recipe |
| GET | `/api/v1/recipes/spoonacular/search` | Authenticated | Search Spoonacular recipes |
| POST | `/api/v1/mealplan` | Authenticated | Save or update meal plan |
| GET | `/api/v1/mealplan/:userId` | Owner or Admin | Get latest meal plan for a user |
| DELETE | `/api/v1/mealplan/:id` | Authenticated | Delete meal plan by ID |
| PATCH | `/api/v1/mealplan/:id/grocery` | Authenticated | Tick/untick grocery item |
| GET | `/api/v1/admin/dashboard` | Admin | Dashboard statistics |
| GET | `/api/v1/admin/users` | Admin | Get all users |
| DELETE | `/api/v1/admin/users/:id` | Admin | Delete a user |
| PATCH | `/api/v1/admin/users/:id/role` | Admin | Change a user's role |

---

## 8. Detailed Endpoint Reference

## 8.1 Health Check

### GET `/api/health`
Returns a simple status message.

#### Response `200 OK`
```json
{
  "success": true,
  "message": "🌿 Balanced Bites API is running!"
}
```

---

## 8.2 Authentication Endpoints

### POST `/api/v1/auth/register`
Registers a new user account.

#### Request body
```json
{
  "name": "Alex",
  "email": "alex@example.com",
  "password": "strongpassword123"
}
```

#### Success response `201 Created`
```json
{
  "success": true,
  "accessToken": "jwt-access-token",
  "refreshToken": "jwt-refresh-token",
  "user": {
    "id": "661111111111111111111111",
    "name": "Alex",
    "email": "alex@example.com",
    "role": "user",
    "isOnboarded": false
  }
}
```

#### Possible errors
- `400` Name, email and password required
- `409` Email already registered
- `500` Server error

---

### POST `/api/v1/auth/login`
Authenticates an existing user.

#### Request body
```json
{
  "email": "alex@example.com",
  "password": "strongpassword123"
}
```

#### Success response `200 OK`
```json
{
  "success": true,
  "accessToken": "jwt-access-token",
  "refreshToken": "jwt-refresh-token",
  "user": {
    "id": "661111111111111111111111",
    "name": "Alex",
    "email": "alex@example.com",
    "role": "user",
    "isOnboarded": true,
    "profile": {
      "goal": "bulk",
      "diet": ["vegetarian"]
    }
  }
}
```

#### Possible errors
- `400` Email and password required
- `401` Invalid email or password
- `500` Server error

---

### POST `/api/v1/auth/refresh`
Generates a new access token using a valid refresh token.

#### Request body
```json
{
  "refreshToken": "jwt-refresh-token"
}
```

#### Success response `200 OK`
```json
{
  "success": true,
  "accessToken": "new-access-token",
  "refreshToken": "new-refresh-token"
}
```

#### Possible errors
- `401` No refresh token
- `401` Invalid refresh token
- `401` Refresh token invalid or expired

---

### GET `/api/v1/auth/profile`
Returns the currently authenticated user.

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Success response `200 OK`
```json
{
  "success": true,
  "user": {
    "_id": "661111111111111111111111",
    "name": "Alex",
    "email": "alex@example.com",
    "role": "user",
    "profile": {},
    "isOnboarded": false
  }
}
```

#### Possible errors
- `401` Not authorised. No token
- `401` Token invalid or expired
- `401` User no longer exists

---

### PUT `/api/v1/auth/profile`
Updates the authenticated user's profile and marks onboarding as complete.

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Request body
```json
{
  "gender": "male",
  "age": 19,
  "weight": 75,
  "weightUnit": "kg",
  "height": 188,
  "heightUnit": "cm",
  "goal": "bulk",
  "workoutFreq": "4-5 times/week",
  "workoutDuration": "60 mins",
  "diet": ["vegetarian"],
  "allergies": ["nuts"]
}
```

#### Success response `200 OK`
```json
{
  "success": true,
  "user": {
    "_id": "661111111111111111111111",
    "profile": {
      "gender": "male",
      "age": 19,
      "weight": 75,
      "weightUnit": "kg",
      "height": 188,
      "heightUnit": "cm",
      "goal": "bulk",
      "workoutFreq": "4-5 times/week",
      "workoutDuration": "60 mins",
      "diet": ["vegetarian"],
      "allergies": ["nuts"]
    },
    "isOnboarded": true
  }
}
```

#### Possible errors
- `401` Authentication error
- `500` Server error

---

### POST `/api/v1/auth/logout`
Logs out the current user by clearing the saved refresh token.

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Success response `200 OK`
```json
{
  "success": true,
  "message": "Logged out."
}
```

---

## 8.3 Recipe Endpoints

### GET `/api/v1/recipes`
Returns recipes from the local database.

#### Query parameters
| Parameter | Type | Required | Description |
|---|---|---:|---|
| `search` | string | No | Text search on recipe name/tag |
| `goal` | string | No | Goal filter, e.g. `cut`, `bulk`, `maintain` |
| `diet` | string | No | Comma-separated diet filter |
| `page` | number | No | Page number, default `1` |
| `limit` | number | No | Results per page, default `12` |

#### Example
```http
GET /api/v1/recipes?goal=bulk&diet=vegetarian&page=1&limit=12
```

#### Success response `200 OK`
```json
{
  "success": true,
  "total": 24,
  "recipes": [
    {
      "_id": "662222222222222222222222",
      "name": "Chicken Rice Bowl",
      "goal": "bulk",
      "diet": ["gluten-free"]
    }
  ]
}
```

---

### GET `/api/v1/recipes/:id`
Returns one recipe by MongoDB ID.

#### Success response `200 OK`
```json
{
  "success": true,
  "recipe": {
    "_id": "662222222222222222222222",
    "name": "Chicken Rice Bowl"
  }
}
```

#### Possible errors
- `404` Recipe not found
- `500` Server error

---

### POST `/api/v1/recipes`
Creates a new recipe.

**Access:** Admin only

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Request body
```json
{
  "name": "Chicken Rice Bowl",
  "description": "High-protein balanced meal",
  "emoji": "🍗",
  "tag": "High Protein",
  "calories": 540,
  "prepTime": "25 min",
  "servings": 2,
  "goal": "bulk",
  "diet": ["gluten-free"],
  "allergies": [],
  "ingredients": [
    { "name": "Chicken breast", "amount": "200g" },
    { "name": "Rice", "amount": "150g" }
  ],
  "steps": [
    "Cook the rice",
    "Grill the chicken",
    "Assemble the bowl"
  ],
  "nutrition": {
    "protein": 42,
    "carbs": 48,
    "fat": 12,
    "fibre": 4
  },
  "color": "#E8F5E9"
}
```

#### Success response `201 Created`
```json
{
  "success": true,
  "recipe": {
    "_id": "662222222222222222222222",
    "name": "Chicken Rice Bowl",
    "createdBy": "661111111111111111111111"
  }
}
```

#### Possible errors
- `401` Authentication error
- `403` Admin access required
- `400` Validation error

---

### PUT `/api/v1/recipes/:id`
Updates an existing recipe.

**Access:** Admin only

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Request body
Any updatable recipe fields.

#### Success response `200 OK`
```json
{
  "success": true,
  "recipe": {
    "_id": "662222222222222222222222",
    "name": "Updated Chicken Rice Bowl"
  }
}
```

#### Possible errors
- `404` Recipe not found
- `400` Validation error
- `403` Admin access required

---

### DELETE `/api/v1/recipes/:id`
Deletes a recipe.

**Access:** Admin only

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Success response `204 No Content`

---

### GET `/api/v1/recipes/spoonacular/search`
Searches the Spoonacular external API and returns simplified recipe cards.

**Access:** Authenticated users

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Query parameters
| Parameter | Type | Required | Description |
|---|---|---:|---|
| `query` | string | No | Search keyword, default `healthy` |
| `diet` | string | No | Optional Spoonacular diet filter |
| `number` | number | No | Number of results, default `12` |

#### Example
```http
GET /api/v1/recipes/spoonacular/search?query=chicken&diet=vegetarian&number=12
```

#### Success response `200 OK`
```json
{
  "success": true,
  "recipes": [
    {
      "sourceId": "716429",
      "name": "Pasta with Garlic",
      "emoji": "🍽️",
      "tag": "lunch",
      "calories": 520,
      "prepTime": "30 min",
      "diet": ["vegetarian"],
      "color": "#E8F5E9"
    }
  ]
}
```

#### Possible errors
- `401` Authentication error
- `500` Spoonacular error

---

## 8.4 Meal Plan Endpoints

### POST `/api/v1/mealplan`
Creates or updates the authenticated user's meal plan for a given week.

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Request body
```json
{
  "weekStart": "2026-04-06T00:00:00.000Z",
  "slots": [
    {
      "day": "Mon",
      "meal": "Breakfast",
      "recipe": "662222222222222222222222"
    },
    {
      "day": "Mon",
      "meal": "Lunch",
      "recipe": "662222222222222222222223"
    }
  ]
}
```

#### Notes
- The backend collects ingredients from the selected recipes
- A grocery list is generated automatically from those ingredients
- If a plan for the same week already exists, it is updated

#### Success response `200 OK`
```json
{
  "success": true,
  "plan": {
    "_id": "663333333333333333333333",
    "userId": "661111111111111111111111",
    "weekStart": "2026-04-06T00:00:00.000Z",
    "slots": [],
    "groceryList": []
  }
}
```

---

### GET `/api/v1/mealplan/:userId`
Returns the most recent meal plan for a user.

#### Access
- Same user as `:userId`
- Or admin

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Success response `200 OK`
```json
{
  "success": true,
  "plan": {
    "_id": "663333333333333333333333",
    "userId": "661111111111111111111111",
    "groceryList": [
      {
        "name": "Chicken breast",
        "amount": "200g",
        "category": "General",
        "checked": false
      }
    ]
  }
}
```

#### Possible errors
- `403` Not authorised
- `404` No meal plan found
- `500` Server error

---

### DELETE `/api/v1/mealplan/:id`
Deletes a meal plan by ID.

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Success response `204 No Content`

---

### PATCH `/api/v1/mealplan/:id/grocery`
Updates the checked status of one grocery list item inside a meal plan.

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Request body
```json
{
  "itemIndex": 0,
  "checked": true
}
```

#### Success response `200 OK`
```json
{
  "success": true,
  "groceryList": [
    {
      "name": "Chicken breast",
      "amount": "200g",
      "category": "General",
      "checked": true
    }
  ]
}
```

#### Possible errors
- `404` Plan not found
- `500` Server error

---

## 8.5 Admin Endpoints

### GET `/api/v1/admin/dashboard`
Returns admin dashboard statistics and recent users.

**Access:** Admin only

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Success response `200 OK`
```json
{
  "success": true,
  "stats": {
    "totalUsers": 50,
    "totalRecipes": 120,
    "totalPlans": 37
  },
  "recentUsers": [
    {
      "name": "Alex",
      "email": "alex@example.com",
      "role": "user",
      "isOnboarded": true,
      "createdAt": "2026-04-06T10:00:00.000Z"
    }
  ]
}
```

---

### GET `/api/v1/admin/users`
Returns all users.

**Access:** Admin only

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Success response `200 OK`
```json
{
  "success": true,
  "users": [
    {
      "_id": "661111111111111111111111",
      "name": "Alex",
      "email": "alex@example.com",
      "role": "user"
    }
  ]
}
```

---

### DELETE `/api/v1/admin/users/:id`
Deletes a user.

**Access:** Admin only

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Success response `204 No Content`

---

### PATCH `/api/v1/admin/users/:id/role`
Changes a user's role.

**Access:** Admin only

#### Headers
```http
Authorization: Bearer <accessToken>
```

#### Request body
```json
{
  "role": "admin"
}
```

#### Success response `200 OK`
```json
{
  "success": true,
  "user": {
    "_id": "661111111111111111111111",
    "name": "Alex",
    "role": "admin"
  }
}
```

---

## 9. Example Environment Variables

Example backend `.env` values:
```env
PORT=5000
NODE_ENV=development
MONGO_URI=mongodb+srv://your-connection-string
JWT_SECRET=your-access-token-secret
JWT_REFRESH_SECRET=your-refresh-token-secret
CLIENT_URL=http://localhost:5173
SPOONACULAR_API_KEY=your-spoonacular-key
```

Example frontend `.env` value:
```env
VITE_API_URL=http://localhost:5000/api/v1
```

---

## 10. Example Test Requests

### Register
```bash
curl -X POST http://localhost:5000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Alex","email":"alex@example.com","password":"strongpassword123"}'
```

### Login
```bash
curl -X POST http://localhost:5000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alex@example.com","password":"strongpassword123"}'
```

### Get profile
```bash
curl http://localhost:5000/api/v1/auth/profile \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Get recipes
```bash
curl "http://localhost:5000/api/v1/recipes?goal=bulk&diet=vegetarian&page=1&limit=12"
```

### Spoonacular search
```bash
curl "http://localhost:5000/api/v1/recipes/spoonacular/search?query=chicken&number=12" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

---

## 11. Limitations and Notes

- The current implementation provides **OpenAPI-compatible documentation** through the accompanying `openapi.yaml` file, but Swagger UI is **not yet wired into the Express server**.
- Some endpoints return `204 No Content` on delete, so no JSON body is returned.
- The frontend currently auto-refreshes expired access tokens using the stored refresh token.
- Spoonacular search returns a simplified recipe format rather than the full raw external API response.
- This document reflects the backend routes and controllers currently present in the submitted project code.

---

## 12. Suggested Submission Wording

You can describe this in your report like this:

> The backend API was documented using an OpenAPI/Swagger-style specification and a written endpoint reference.  
> The documentation covers authentication, recipes, meal planning, admin functionality, request/response formats, and security requirements.
