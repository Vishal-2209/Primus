## 1. What is an API?

API stands for **Application Programming Interface**. An API is a defined interface and contract through which one software application or component can interact with another software application or component.

In simple terms:

> API defines how one application can request data or functionality from another application.

For example, a banking application may need to verify a customer's PAN details using a government service. The bank does not directly access the government's database. Instead, it sends a request to an API exposed by the government service.

```text
Bank Application
       |
       | API Request
       v
Government API
       |
       v
Government System
       |
       | API Response
       v
Bank Application
```

The API defines things such as:

- Where the request should be sent
- Which HTTP method should be used
- What data should be sent
- How authentication should work
- What format the response will have
- How errors are represented

## 2. API vs REST API

API is a general concept.

REST is an architectural style used to design APIs.

Therefore:

> Every REST API is an API, but every API is not a REST API.

Other API approaches include:

- REST
    
- SOAP
    
- GraphQL
    
- gRPC
    

## 3. What is REST?

REST stands for **Representational State Transfer**.

- REST is an architectural style for designing networked applications.
- REST is not a programming language and it is not a protocol.
- REST APIs commonly use HTTP for communication.
- A REST API generally exposes resources through URLs and uses standard HTTP methods to operate on those resources.

Example resources:

```text
/users
/products
/orders
/cases
/lawyers
```

A specific resource can be identified using a path parameter:

```text
/users/42
```

Here, `42` identifies a particular user.

## 4. Client Server Architecture

One of the fundamental REST constraints is the separation between the client and the server. The client is responsible for the user-facing application and making requests. The server is responsible for processing requests, executing business logic, interacting with databases and returning responses.

```text
Client
  |
  | HTTP Request
  v
Server
  |
  | Database Query
  v
Database
```

For example:

```text
React Application
       |
       | HTTP Request
       v
FastAPI Backend
       |
       | SQL Query
       v
PostgreSQL
```

The React application should not directly access the PostgreSQL database.

Instead:

```text
Client -> API -> Database
```

This provides separation of concerns and allows the backend to control authentication, authorization, validation and business logic.

## 5. HTTP Methods

REST APIs commonly use HTTP methods to indicate the intended operation.

| Method | Purpose                                 |
| ------ | --------------------------------------- |
| GET    | Retrieve data                           |
| POST   | Create a new resource                   |
| PUT    | Completely replace an existing resource |
| PATCH  | Partially update an existing resource   |
| DELETE | Delete a resource                       |

Example:

```http
GET /users/42
```

Retrieve user 42.

```http
POST /users
```

Create a new user.

```http
PUT /users/42
```

Completely replace user 42.

```http
PATCH /users/42
```

Partially update user 42.

```http
DELETE /users/42
```

Delete user 42.

## 6. HTTP Request

When a client communicates with a server using HTTP, it sends an HTTP request.

An HTTP request can contain:

```text
HTTP Request
|
|-- Method
|-- URL
|-- Headers
|-- Body
```

Example:

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer <token>

{
    "name": "Vishal",
    "email": "vishal@va7.dev"
}
```

## 7. URL

A URL identifies where the request should be sent.

Example:

```text
https://api.example.com/users/42
```

Basic structure:

```text
https://api.example.com/users/42
|       |               |
|       |               |
Protocol Host            Path
```

Query parameters can be used to provide additional information.

Example:

```text
/users?page=2&limit=10
```

Here:

```text
page = 2
limit = 10
```

are query parameters.

## 8. HTTP Headers

Headers contain metadata about the HTTP request or response.

Example:

```http
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
```

### Content-Type

`Content-Type` tells the server what format the request body is using.

```http
Content-Type: application/json
```

This means that the request body contains JSON data.

### Authorization

The `Authorization` header is commonly used to send authentication credentials.

Example:

```http
Authorization: Bearer <token>
```

### Accept

`Accept` tells the server which response format the client can handle or prefers.

Example:

```http
Accept: application/json
```

Important distinction:

> Headers contain metadata, while the body generally contains the actual payload.

## 9. HTTP Request Body

The request body contains data sent to the server.

It is commonly used with methods such as POST, PUT and PATCH.

Example:

```http
POST /users
Content-Type: application/json

{
    "name": "Vishal",
    "age": 21
}
```

The JSON object is the request body.

GET requests generally use query parameters instead of a request body.

## 10. JSON

JSON stands for **JavaScript Object Notation**.

JSON is a lightweight data interchange format commonly used by REST APIs.

Example:

```json
{
    "name": "Vishal",
    "age": 21,
    "active": true,
    "skills": [
        "Python",
        "FastAPI"
    ]
}
```

JSON supports:

- String
- Number
- Boolean
- Null
- Object
- Array

JSON is popular because it is human-readable and can be easily parsed by most programming languages.

Important:

> REST does not require JSON. JSON is simply the most commonly used representation format in modern REST APIs.

## 11. HTTP Response

After processing a request, the server sends an HTTP response.

An HTTP response generally contains:

```text
HTTP Response
|
|-- Status Code
|-- Headers
|-- Body
```

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": 42,
    "name": "Vishal"
}
```

## 12. HTTP Status Codes

HTTP status codes indicate the result of an HTTP request.

```text
1xx -> Informational
2xx -> Success
3xx -> Redirection
4xx -> Client Error
5xx -> Server Error
```

Important status codes:

|Status Code|Meaning|
|---|---|
|200|OK|
|201|Created|
|204|No Content|
|400|Bad Request|
|401|Unauthorized|
|403|Forbidden|
|404|Not Found|
|409|Conflict|
|422|Unprocessable Content|
|500|Internal Server Error|
|503|Service Unavailable|

### 401 vs 403

These two are commonly confused.

**401 Unauthorized**
Usually means the client is not properly authenticated.

Example:

```text
Missing token
Invalid token
Expired token
```

**403 Forbidden**
Means the client is authenticated but does not have permission to perform the requested operation.

Example:

```text
Authenticated User
        |
        | Try to access admin-only endpoint
        v
      403
```

## 13. Authentication vs Authorization

Authentication and authorization are different concepts.

### Authentication

Authentication answers:

> Who are you?

Examples:

- Username and password
- JWT
- API key
- OAuth access token
- Client certificates
### Authorization

Authorization answers:

> What are you allowed to do?

Example:

```text
User
  -> Can read own profile

Admin
  -> Can read all profiles
  -> Can delete users
```

The typical flow is:

```text
Authentication
      |
      v
Who are you?
      |
      v
Authorization
      |
      v
What are you allowed to access?
```

An API key does not automatically mean that the client is authorized to perform every operation. Authorization depends on the permissions, scopes, roles or policies associated with the credentials.

## 14. Statelessness

Statelessness is one of the important REST constraints.

It means that every request should contain all the information necessary for the server to understand and process that request. The server should not depend on information stored from a previous request.

Example:

```http
GET /profile
Authorization: Bearer <token>
```

```http
GET /orders
Authorization: Bearer <token>
```

The second request should contain the necessary authentication information itself. The server should not need to remember that the first request already authenticated the client.

## 15. REST Architectural Constraints

REST is based on six major architectural constraints:

1. Client-Server
2. Stateless
3. Cacheable
4. Uniform Interface
5. Layered System
6. Code on Deman

The first five are core concepts to understand. Code on Demand is optional.

## 16. Complete REST API Request Flow

A typical REST API request can be understood as:

```text
Client
  |
  | HTTP Request
  | Method
  | URL
  | Headers
  | Body
  v
FastAPI Server
  |
  | Authentication
  | Authorization
  | Validation
  | Business Logic
  v
Database
  |
  | Data
  v
FastAPI Server
  |
  | HTTP Response
  | Status Code
  | Headers
  | Body
  v
Client
```

Example:

```http
POST /users
Authorization: Bearer <token>
Content-Type: application/json

{
    "name": "Vishal",
    "email": "vishal@example.com"
}
```

Server processing:

```text
Receive Request
      |
      v
Authenticate
      |
      v
Authorize
      |
      v
Validate Request Data
      |
      v
Execute Business Logic
      |
      v
Store Data in Database
      |
      v
Generate Response
```

Response:

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
    "id": 101,
    "name": "Vishal",
    "email": "vishal@example.com"
}
```

## 17. Important Interview Distinctions

### API vs REST API

```text
API
-> General interface for software communication

REST API
-> API designed according to REST architectural principles
```
### Header vs Body

```text
Header
-> Metadata

Body
-> Actual request or response payload
```
### Authentication vs Authorization

```text
Authentication
-> Who are you?

Authorization
-> What are you allowed to do?
```
### PUT vs PATCH

```text
PUT
-> Complete replacement

PATCH
-> Partial modification
```
### 401 vs 403

```text
401
-> Authentication problem

403
-> Authorization/permission problem
```
### REST vs JSON

```text
REST
-> Architectural style

JSON
-> Data representation format
```
### Client vs Server

```text
Client
-> Requests data/services

Server
-> Processes requests and provides data/services
```
## 18. One Line Mental Model

For quick revision:

```text
Client
  |
  | HTTP Request
  | Method + URL + Headers + Body
  v
REST API
  |
  | Authentication + Authorization
  | Validation + Business Logic
  v
Database
  |
  v
REST API
  |
  | HTTP Response
  | Status Code + Headers + Body
  v
Client
```

**Core idea:**
> A REST API is an HTTP-based interface where clients interact with server-side resources using standardized methods, representations and architectural constraints.