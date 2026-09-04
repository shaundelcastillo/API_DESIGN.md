# Library Management System API Design

## 1. Overview
This API provides a RESTful backend interface for managing a Library Management System. It handles member account management, catalog browsing, loan/borrowing lifecycles, and category classification following standard REST principles.

## 2. Base URL
```text
Production: [https://api.library.com/v1](https://api.library.com/v1)
Development: http://localhost:3000/api
3. Resources & Properties
1. members
id (integer, auto-generated, primary key)

first_name (string, required)

last_name (string, required)

email (string, unique, required)

phone (string)

created_at (timestamp)

updated_at (timestamp)

2. books
id (integer, auto-generated, primary key)

title (string, required)

author (string, required)

isbn (string, unique, required)

published_year (integer)

category_id (integer, foreign key referencing categories)

available_copies (integer, default: 1)

created_at (timestamp)

updated_at (timestamp)

3. loans
id (integer, auto-generated, primary key)

member_id (integer, foreign key referencing members)

book_id (integer, foreign key referencing books)

loan_date (timestamp)

due_date (timestamp)

returned_at (timestamp, nullable)

status (string: "active", "returned", "overdue")

created_at (timestamp)

updated_at (timestamp)

4. categories
id (integer, auto-generated, primary key)

name (string, unique, required)

description (string)

created_at (timestamp)

updated_at (timestamp)

4. Endpoints & CRUD Design
Standard CRUD Endpoints
Members (/api/members)
GET /api/members - Get all members

GET /api/members/:id - Get a member by ID

POST /api/members - Create a new member

PUT /api/members/:id - Replace member profile completely

PATCH /api/members/:id - Partially update member profile

DELETE /api/members/:id - Delete a member account

Books (/api/books)
GET /api/books - Get all books

GET /api/books/:id - Get one book by ID

POST /api/books - Add a new book to the catalog

PUT /api/books/:id - Replace book details completely

PATCH /api/books/:id - Partially update book details

DELETE /api/books/:id - Remove a book from the catalog

Loans (/api/loans)
GET /api/loans - Get all loans

GET /api/loans/:id - Get details of a specific loan record

POST /api/loans - Borrow a book (create a new loan)

PUT /api/loans/:id - Replace loan record details completely

PATCH /api/loans/:id - Update loan status

DELETE /api/loans/:id - Delete a loan record

Categories (/api/categories)
GET /api/categories - Get all book categories

GET /api/categories/:id - Get a category by ID

POST /api/categories - Create a new book category

PUT /api/categories/:id - Replace category details

PATCH /api/categories/:id - Partially update a category

DELETE /api/categories/:id - Delete a category

Nested Resource Endpoints
GET /api/members/:id/loans - Get all loans for a specific member

POST /api/members/:id/loans - Borrow a book specifically for a member

GET /api/categories/:id/books - Get all books assigned to a specific category

GET /api/books/:id/loans - Get borrowing history for a specific book

5. Request & Response Examples
1. POST /api/members (Create Member)
Request Body:

JSON
{
  "first_name": "Juan",
  "last_name": "Dela Cruz",
  "email": "juan.delacruz@example.com",
  "phone": "+639171234567"
}
Success Response (201 Created):

JSON
{
  "id": 1,
  "first_name": "Juan",
  "last_name": "Dela Cruz",
  "email": "juan.delacruz@example.com",
  "phone": "+639171234567",
  "created_at": "2026-09-04T10:00:00Z",
  "updated_at": "2026-09-04T10:00:00Z"
}
2. GET /api/members/:id (Get Member Details)
Success Response (200 OK):

JSON
{
  "id": 1,
  "first_name": "Juan",
  "last_name": "Dela Cruz",
  "email": "juan.delacruz@example.com",
  "phone": "+639171234567",
  "created_at": "2026-09-04T10:00:00Z",
  "updated_at": "2026-09-04T10:00:00Z"
}
Error Response (404 Not Found):

JSON
{
  "error": {
    "code": "MEMBER_NOT_FOUND",
    "message": "Member with ID 1 does not exist"
  }
}
3. POST /api/loans (Borrow a Book)
Request Body:

JSON
{
  "member_id": 1,
  "book_id": 5
}
Success Response (201 Created):

JSON
{
  "id": 101,
  "member_id": 1,
  "book_id": 5,
  "loan_date": "2026-09-04T10:30:00Z",
  "due_date": "2026-09-18T10:30:00Z",
  "returned_at": null,
  "status": "active",
  "created_at": "2026-09-04T10:30:00Z",
  "updated_at": "2026-09-04T10:30:00Z"
}
Error Response (409 Conflict - Out of stock):

JSON
{
  "error": {
    "code": "BOOK_NOT_AVAILABLE",
    "message": "This book has 0 available copies for borrowing"
  }
}
4. PATCH /api/books/:id (Update Book Availability)
Request Body:

JSON
{
  "available_copies": 3
}
Success Response (200 OK):

JSON
{
  "id": 5,
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "available_copies": 3,
  "updated_at": "2026-09-04T11:00:00Z"
}
5. DELETE /api/books/:id (Delete Book)
Success Response (200 OK):

JSON
{
  "message": "Book with ID 5 was successfully deleted"
}
6. Status Code Mappings
GET /api/books/:id

Success (200 OK): Book details retrieved.

Not Found (404 Not Found): Specified ID does not exist.

Server Error (500 Internal Server Error): Database query failure.

POST /api/members

Created (201 Created): Member account created successfully.

Bad Request (400 Bad Request): Missing required fields.

Conflict (409 Conflict): Email already registered.

POST /api/loans

Created (201 Created): Loan processed successfully.

Bad Request (400 Bad Request): Invalid payload structure.

Not Found (404 Not Found): Member or book ID missing.

Conflict (409 Conflict): No copies remaining for checkout.

DELETE /api/books/:id

Success (200 OK): Book deleted successfully.

Not Found (404 Not Found): Book ID does not exist.

Conflict (409 Conflict): Cannot delete book with active loans attached.

7. Special Scenarios & Edge Cases
Scenario 1: Borrowing an unavailable book (0 copies remaining)

Status Code: 409 Conflict

Error Code: "BOOK_NOT_AVAILABLE"

Error Message: "This book currently has no available copies for borrowing."

Scenario 2: Returning an already-returned loan

Status Code: 400 Bad Request

Error Code: "LOAN_ALREADY_CLOSED"

Error Message: "This loan record has already been marked as returned."

Scenario 3: Deleting a book that has active loans attached

Status Code: 409 Conflict

Error Code: "CANNOT_DELETE_ACTIVE_LOAN_BOOK"

Error Message: "Cannot delete book because there are active loans associated with it."

Scenario 4: Registering a member with a duplicate email address

Status Code: 409 Conflict

Error Code: "EMAIL_ALREADY_EXISTS"

Error Message: "A member with this email address is already registered."

8. Advanced Features (Challenges)
Challenge 1: Query Parameters for Filtering & Sorting
1. Books Search & Filtering
Endpoint: GET /api/books
Query Parameters:

?search=clean              → Search books with "clean" in title or author

?category_id=3            → Filter books belonging to Category ID 3

?sort=published_year       → Sort by publication year

?order=desc                → Newest first (asc/desc)

?page=1&limit=10           → Pagination (First page, 10 items per page)

Full URL: GET /api/books?search=clean&category_id=3&sort=published_year&order=desc&page=1&limit=10

2. Loans Filtering
Endpoint: GET /api/loans
Query Parameters:

?status=overdue            → Filter loans by status (active, returned, or overdue)

?member_id=1              → Filter loans assigned to Member ID 1

?sort=due_date             → Sort by due date

?order=asc                 → Earliest due date first (asc/desc)

?page=1&limit=10           → Pagination (First page, 10 items per page)

Full URL: GET /api/loans?status=overdue&member_id=1&sort=due_date&order=asc&page=1&limit=10

Challenge 2: API Versioning
Current Version: v1
New Version: v2

Breaking Change: Replace flat "author" string on books with structured "author_id" object reference and remove "available_copies" field.
Migration Strategy: Maintain concurrent support for both /v1/books and /v2/books endpoints for 6 months.
Deprecation Timeline: Announce v1 deprecation on 2026-10-01, sunset and remove v1 endpoints on 2027-04-01.