# School Management API - Backend System in Go

This project is a backend system for a school management API, built using Go (Golang), the Gin framework, GORM ORM, and PostgreSQL.

## Table of Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [API Planning Adherence](#api-planning-adherence)
- [Project Structure](#project-structure)
- [Setup and Installation](#setup-and-installation)
  - [Prerequisites](#prerequisites)
  - [Environment Variables](#environment-variables)
  - [Running with Docker (Recommended)](#running-with-docker-recommended)
  - [Running Locally](#running-locally)
- [Database Migrations](#database-migrations)
- [API Endpoints Documentation](#api-endpoints-documentation)
- [Testing](#testing)
- [Further Improvements](#further-improvements)

## Project Overview

This API allows administrative staff to manage students, teachers, staff members, and executives. It includes functionalities for adding, modifying, deleting, and listing these entities, along with authentication, class management, security features like rate limiting, and password management.

## Features

- **User Management**:
    - CRUD operations for Executives.
    - CRUD operations for Students.
    - (Foundation for Teachers and Staff can be extended similarly).
- **Authentication**:
    - User registration (implicitly via creating an executive/student).
    - JWT-based Login (`POST /auth/login`).
    - Logout (client-side token removal; server-side stateless).
    - Protected endpoints using JWT middleware.
- **Class Management**:
    - CRUD for Classes.
    - List students in a class.
    - Get student count for a class.
- **Security**:
    - Password hashing (bcrypt).
    - Basic rate limiting.
    - Stubs for password reset mechanisms.
    - User deactivation (soft delete via `is_active` flag).
- **Database**:
    - PostgreSQL as the relational database.
    - GORM as the ORM.

## API Planning Adherence

The implementation covers key aspects of the provided API plan, focusing on:

**Executives:**
- `GET /execs`: Get list of executives
- `POST /execs`: Add a new executive
- `GET /execs/{id}`: Get a specific executive
- `PATCH /execs/{id}`: Modify a specific executive
- `DELETE /execs/{id}`: Delete a specific executive
- `POST /auth/login`: Login (generic for users, including executives)
- *(Logout, Forgot Password, Reset Password are conceptualized/stubbed)*

**Students:**
- `GET /students`: Get list of students
- `POST /students`: Add a new student
- `GET /students/{id}`: Get a specific student
- `PATCH /students/{id}`: Modify a specific student
- `DELETE /students/{id}`: Delete a specific student

**Class Management:**
- Total count of students in a class and class teacher details.
- List of all students in a class with class teacher details.

*(Bulk modifications, and full CRUD for teachers/staff can be built upon the provided structure.)*

## Project Structure (Layered Pattern)

This project utilizes a **Layered Architecture Pattern**. This pattern organizes the codebase into distinct layers, each with a specific responsibility. This promotes separation of concerns, modularity, testability, and maintainability.

The layers are:

1.  **`cmd/server/main.go`**: The entry point of the application. Initializes configurations, database connections, router, and starts the HTTP server.
2.  **`internal/config`**: Handles application configuration, typically loaded from environment variables or configuration files.
3.  **`internal/domain`**: Contains the core business logic and entities (structs like `User`, `Class`) and interfaces for repositories. This layer is the heart of the application and should be independent of other layers like transport or database specifics.
4.  **`internal/repository`**: Implements the data access logic defined by interfaces in the `domain` layer. This layer interacts directly with the database (e.g., using GORM). It isolates database-specific code.
5.  **`internal/app` (or `service`)**: Contains the application-specific business logic (use cases). It orchestrates operations by using repositories to fetch/store data and applying business rules. It acts as an intermediary between the API handlers and the domain/repositories.
6.  **`internal/api` (or `handler`)**: Handles incoming HTTP requests, validates input, calls the appropriate services in the `app` layer, and formats HTTP responses. This layer uses the Gin framework for routing and request/response handling.
7.  **`internal/middleware`**: Contains Gin middleware for concerns like authentication (JWT validation), rate limiting, logging, etc.
8.  **`internal/util`**: Provides utility functions used across different parts of the application (e.g., password hashing, token generation).
9.  **`migrations`**: (Optional for GORM auto-migrate, but good practice) SQL files for database schema changes.
10. **`tests`**: Contains test files, with subdirectories for unit, integration, or end-to-end (E2E) tests.

**Reasons for choosing the Layered Pattern:**

-   **Separation of Concerns**: Each layer has a well-defined responsibility, making the system easier to understand, develop, and maintain.
-   **Modularity**: Components within a layer can be modified or replaced without significantly impacting other layers (e.g., changing the database or ORM primarily affects the repository layer).
-   **Testability**: Individual layers can be tested in isolation. For example, services can be tested by mocking repositories.
-   **Scalability**: Clear separation makes it easier to scale different parts of the application independently.
-   **Maintainability**: Code is organized logically, making it easier for new developers to understand the codebase and for existing developers to make changes.

## Setup and Installation

### Prerequisites

-   Go (version 1.20 or higher)
-   PostgreSQL (or Docker to run PostgreSQL)
-   Git

### Environment Variables

Create a `.env` file in the root directory by copying `.env.example` and fill in the required values:

```sh
cp .env.example .env
