# OneID

# Digital Signature Tool

This project is a secure, server-side digital signature tool designed to generate and manage unique, immutable digital signatures for users. Built with a focus on security, performance, and reliability, the system uses Rust for the backend, Axum as the web framework, and PostgreSQL for the database.

## Table of Contents

  - [Features](https://www.google.com/search?q=%23features)
  - [System Design](https://www.google.com/search?q=%23system-design)
  - [Prerequisites](https://www.google.com/search?q=%23prerequisites)
  - [Getting Started](https://www.google.com/search?q=%23getting-started)
  - [Configuration](https://www.google.com/search?q=%23configuration)
  - [Database Schema](https://www.google.com/search?q=%23database-schema)
  - [API Endpoints](https://www.google.com/search?q=%23api-endpoints)
  - [Security Considerations](https://www.google.com/search?q=%23security-considerations)
  - [Contributing](https://www.google.com/search?q=%23contributing)
  - [License](https://www.google.com/search?q=%23license)

## Features

  - **Secure User Authentication:** User registration, login, and session management using strong password hashing with Argon2.
  - **Unique Digital Signatures:** Generates a unique, non-alterable digital signature for each user based on verified identity information.
  - **Data Integrity:** Digital signatures are immutable once created and cannot be changed or deleted.
  - **Identity Verification Integration:** Designed to integrate with a trusted third-party identity verification service (KYC) to handle sensitive data like government IDs and biometrics securely.
  - **Robust Database Management:** Utilizes PostgreSQL for data storage, with `sqlx` for compile-time query checking and migrations.
  - **Async & High-Performance:** Built on Rust's async ecosystem with Axum for a fast and scalable server.
  - **Account Recovery:** Secure password reset and email recovery flows.
  - **Two-Factor Authentication (2FA):** Support for authenticator apps (TOTP) to enhance security.

## System Design

The system follows a microservice-like architecture with a clear separation of concerns:

  - **Web Server (Axum):** Handles all incoming HTTP requests, routes them to the appropriate handlers, and serves API responses.
  - **Authentication Layer:** Manages user creation, login, session validation, and account recovery. Passwords are never stored in plain text.
  - **Digital Signature Service:** The core of the application, responsible for receiving validated identity information, generating a unique hash, and storing it immutably. It includes a check to prevent hash collisions.
  - **Database (PostgreSQL):** The centralized data store for users and their digital signatures.
  - **Third-Party Integrations:** The system is designed to interact with external services for email delivery and identity verification (KYC).

## Prerequisites

Before you begin, ensure you have the following installed:

  - **Rust:** The Rust programming language and Cargo package manager.
  - **PostgreSQL:** A running PostgreSQL database instance.
  - **Just:** A command runner similar to `make`, for simplifying development tasks (optional but recommended).

## Getting Started

1.  **Clone the repository:**

    ```bash
    git clone https://github.com/your-username/digital-signature-tool.git
    cd digital-signature-tool
    ```

2.  **Set up the environment:**
    Create a `.env` file in the project root based on the provided `.env.example`.

3.  **Run database migrations:**
    `sqlx-cli` is required for running migrations. If you don't have it, install it with `cargo install sqlx-cli`.

    ```bash
    sqlx migrate run
    ```

    Alternatively, if you're using `just`:

    ```bash
    just migrate
    ```

4.  **Run the server:**

    ```bash
    cargo run
    ```

    If you're using `just`:

    ```bash
    just run
    ```

The server will start on the address specified in your `.env` file (e.g., `http://127.0.0.1:8000`).

## Configuration

The project is configured via environment variables. Create a `.env` file with the following variables:

```ini
# Database
DATABASE_URL=postgres://user:password@localhost:5432/digital_signature_db

# Server
SERVER_BIND_ADDRESS=127.0.0.1:8000
SECRET_KEY=a_32_byte_secret_key_for_jwt_tokens

# Email Service (for password reset, account verification)
EMAIL_SENDER=no-reply@example.com
SMTP_SERVER=smtp.example.com
SMTP_PORT=587
SMTP_USERNAME=your_email_user
SMTP_PASSWORD=your_email_password
```

## Database Schema

The core database schema consists of the following tables:

  - **`users`:**

      - `id`: `UUID` (Primary Key)
      - `username`: `VARCHAR(255)` (Unique)
      - `email`: `VARCHAR(255)` (Unique)
      - `password_hash`: `VARCHAR(255)` (Hashed password)
      - `created_at`: `TIMESTAMPTZ`
      - `updated_at`: `TIMESTAMPTZ`
      - `is_active`: `BOOLEAN` (For account deletion/deactivation)

  - **`digital_signatures`:**

      - `id`: `UUID` (Primary Key)
      - `user_id`: `UUID` (Foreign Key to `users.id`, Unique)
      - `signature_hash`: `VARCHAR(255)` (Unique cryptographic hash)
      - `created_at`: `TIMESTAMPTZ`

  - **`password_reset_tokens`:**

      - `id`: `UUID` (Primary Key)
      - `user_id`: `UUID` (Foreign Key to `users.id`)
      - `token`: `VARCHAR(255)` (Unique, time-limited token)
      - `expires_at`: `TIMESTAMPTZ`

## API Endpoints

A detailed API documentation will be provided, but here are the main endpoints:

  - `POST /api/register` - Create a new user account.
  - `POST /api/login` - Authenticate a user and create a session.
  - `POST /api/forgot-password` - Initiate a password reset.
  - `POST /api/reset-password` - Reset the password with a valid token.
  - `POST /api/create-signature` - Create and store a new digital signature for the authenticated user.
  - `GET /api/me` - Get the current user's profile information.

## Security Considerations

  - **Secure by Design:** The project adheres to a Secure Software Development Lifecycle (SSDLC) with a focus on threat modeling, secure coding, and vulnerability testing.
  - **No PII Storage:** This system is explicitly designed **not to store raw PII** like government IDs or biometric data. It relies on trusted third-party services for verification.
  - **HTTPS is Mandatory:** The server should always be run behind a reverse proxy (like Nginx or Caddy) to handle SSL/TLS termination and ensure all traffic is encrypted.
  - **Input Sanitization:** All user inputs are sanitized to prevent common web vulnerabilities like Cross-Site Scripting (XSS) and SQL Injection. `sqlx`'s prepared statements handle SQL injection protection automatically.

## Contributing

We welcome contributions\! Please feel free to open issues or submit pull requests.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.
