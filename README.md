# Dating App Backend

Hello there,

I would like to point out that this application is still in the development phase and not ready for production. I have introduced some basic concepts to demonstrate the general idea. For example, I did not implement a caching solution or use Elasticsearch, which could enhance the search functionality significantly. Also, the unit testing is not exhaustive and does not cover all edge cases, but it captures the main concepts I wanted to showcase.
Also, initially I considered implementing a DDD approach, but after some deliberation, I opted for a structure that, although less focused on domain separation, centralized the logic and repository layers. This approach not only standardized the components but also introduced some architectural challenges that I believe are beneficial from an interview perspective

hopefully you will find this solution useful.

## Project Overview

This backend is structured to support a dating application, featuring user management, authentication, and matching based on location and preferences.

https://github.com/a-berahman/dating-app/assets/62759025/01178e9f-f425-4521-9d15-b240532981b1

## Architecture
The codebase is organized following a modified clean architecture to ensure separation of concerns and scalability:

- cmd/: Contains the application entry point and setup configurations
- internal/: Core application logic, domain-specific code
- repository/: Data access layer and interfaces for database interaction
- logic/: Business logic layer where core application rules are implemented
- handlers/: HTTP server and routing definitions
- pkg/: General utilities and middleware functions
- constants/: Application wide constants and error definitions
- model/: Data structures that represent domain objects

* for project layout I have used the following link as a reference: https://github.com/golang-standards/project-layout

## Technical Stack
Go: Programming language.
Echo: Web framework for handling HTTP requests.
GORM: ORM library for Go.
PostgreSQL with PostGIS: Database system with support for geographic objects.
Docker and DockerCompose: Containerization of the application and database

## Development Setup

Ensure you have Docker and Go installed on your machine, clone the repository and navigate to the project directory.

## Environment Variables

Set the following environment variables in your .env file or export them in your shell:

- DB_HOST: Database host
- DB_USER: Database user
- DB_PASS: Database password
- DB_NAME: Database name
- DB_PORT: Database por.
- JWT_SECRET_KEY: Secret key for JWT
- JWT_DURATION: Duration for JWT expiration
- LOG_ENV: Logging environment (development or production)

## Building and Running

You can build and run the project using Docker:

```
make docker-compose-up
```

OR

```
docker-compose up --build
```
This command builds the application and starts all services, including the PostGIS database.


### important Notes
The application service (app) depends on the database service (db). It will only start once the database is fully ready and healthy.
This is managed by the depends_on clause with a service_healthy condition in the Docker Compose file. The PostGIS extension might take a bit of time to initialize. To handle this, the database service includes a health check that ensures the database is ready to accept connections before the application starts.


To stop the services, you can use:

```
make docker-compose-down
```

## Testing

Run the following command to execute unit tests:

```
make test
```

## API Endpoints

The server will start on port 8080, and you can access the API at http://localhost:8080/ , here are some CURL commands to interact with the API:

* remember the valid value for gender is MALE or FEMALE

```
# Create a Random User
curl -X POST http://localhost:8080/user/create

# Login
curl -X POST http://localhost:8080/login \
    -H "Content-Type: application/json" \
    -d '{
        "email": "mariliemetz@moore.com",
        "password": "WidMJHF_q51?"
    }'

# Discover Matches
curl -X GET "http://localhost:8080/discover?lat=34.0522&lng=-118.2437&distance=10000&gender=MALE&minAge=18&maxAge=50" \
    -H "Authorization: Bearer YOUR_JWT_TOKEN"

# Swipe on a Profile
curl -X POST http://localhost:8080/swipe \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer YOUR_JWT_TOKEN" \
    -d '{
        "targetUserId": 2,
        "preference": "YES"
    }'

```

## Notes

- The application uses JWT for authentication; make sure to replace YOUR_JWT_TOKEN with a valid token obtained from the login endpoint
- The match discovery algorithm considers user preferences and geographical distance to suggest potential matches



## Example of Match Curls
```

❯ curl -X POST http://localhost:8080/user/create
{"result":{"age":64,"email":"tadward@abernathy.name","gender":"FEMALE","id":6,"name":"Tamara Miller","password":"7sT32u#9UR45"}}
❯ curl -X POST http://localhost:8080/user/create
{"result":{"age":22,"email":"maryryan@kreiger.net","gender":"FEMALE","id":7,"name":"Sid Sauer","password":"hj.xBtDuf!yd"}}

❯ curl -X POST http://localhost:8080/login \
    -H "Content-Type: application/json" \
    -d '{
        "email": "tadward@abernathy.name",
        "password": "7sT32u#9UR45"
    }'
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MTUwNzE1NjIsIlVzZXJJRCI6Nn0.VtD8pBZwMdCdZuutp31Mv1FQ-lhGB5gyPB0yMb2l-oQ"}
❯     curl -X POST http://localhost:8080/login \
    -H "Content-Type: application/json" \
    -d '{
        "email": "maryryan@kreiger.net",
        "password": "hj.xBtDuf!yd"
    }'
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MTUwNzE1NzIsIlVzZXJJRCI6N30.mxEtEHxvJ5UqSDzNCpiDxAeS-sxoE3E37jgTKPu0vR8"}


❯ curl -X POST http://localhost:8080/swipe \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MTUwNzE1NjIsIlVzZXJJRCI6Nn0.VtD8pBZwMdCdZuutp31Mv1FQ-lhGB5gyPB0yMb2l-oQ" \
    -d '{
        "targetUserId": 7,
        "preference": "YES"
    }'
{"results":{"matched":false}}
❯   curl -X POST http://localhost:8080/swipe \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MTUwNzE1NzIsIlVzZXJJRCI6N30.mxEtEHxvJ5UqSDzNCpiDxAeS-sxoE3E37jgTKPu0vR8" \
    -d '{
        "targetUserId": 6,
        "preference": "YES"
    }'
{"results":{"matched":true,"matchID":1}}


```
