# E-Wallet

> Personal finance management platform — bank accounts, investment portfolios, and wealth tracking in one place.

E-Wallet is a full-stack Jakarta EE application that lets users manage their financial life: multiple bank accounts with full transaction history, investment portfolios across stocks, ETFs and cryptocurrencies, and a consolidated wealth dashboard with historical trends and real-time market data.

---

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Seed data](#seed-data)
- [Configuration](#configuration)
- [REST API Reference](#rest-api-reference)
  - [Users](#users)
  - [Accounts](#accounts)
  - [Transactions](#transactions)
  - [Portfolios](#portfolios)
  - [Assets](#assets)
  - [Portfolio Transactions (Trades)](#portfolio-transactions-trades)
  - [Wealth](#wealth)
  - [Admin](#admin)
- [Database](#database)
- [Project Structure](#project-structure)
- [Running Tests](#running-tests)
- [Known Limitations & Roadmap](#known-limitations--roadmap)

---

## Features

### Bank Accounts
- Create and manage multiple bank accounts per user
- Deposit, withdraw, and transfer funds between accounts
- Complete transaction history with timestamps and audit trail
- Balances tracked with high precision (`DECIMAL(19,4)`)

### Investment Portfolios
- Multiple portfolios per user
- Buy and sell **stocks**, **ETFs** and **cryptocurrencies**
- Weighted average cost basis automatically updated on each trade
- Zero-quantity positions cleaned up automatically
- Full trade history per portfolio

### Real-Time Market Data
- Live stock and ETF prices via **Finnhub API**
- Live crypto prices via **CoinGecko API** (no API key required)
- In-memory price cache to reduce external API calls
- Automatic fallback to mock data when APIs are unavailable

### Wealth Dashboard
- Consolidated view: total cash (all accounts) + equity (all portfolios)
- Historical wealth snapshots over time
- Multi-currency support: USD / CHF conversion
- Interactive charts powered by PrimeFaces

### Authentication & Security
- User registration with email validation
- Email + password login
- Session-based authentication
- Route guard via `AuthFilter` — all protected pages redirect to login if unauthenticated
- Optimistic locking on all entities (`@Version`) to prevent concurrent modification conflicts
- Password hashing aligned with OWASP 2024 / NIST SP800-63B guidelines

---

## Architecture

The project follows a classic 3-tier architecture split into three Maven modules deployed on a single Payara instance:

```
┌─────────────────────────────────────────────────────┐
│                    Browser                          │
└───────────────────────┬─────────────────────────────┘
                        │ HTTP
┌───────────────────────▼─────────────────────────────┐
│              Payara Server 6                        │
│                                                     │
│  ┌───────────────────────────────────────────────┐  │
│  │  webapp.war  (JSF / PrimeFaces)               │  │
│  │  - Backing beans (@SessionScoped)             │  │
│  │  - AuthFilter (session guard)                 │  │
│  │  - BackendApiService (REST client)            │  │
│  │  - FinnhubService / CoinGeckoService          │  │
│  └───────────────────┬───────────────────────────┘  │
│                      │ HTTP REST                    │
│  ┌───────────────────▼───────────────────────────┐  │
│  │  webservice.war  (JAX-RS /api)                │  │
│  │  - UserResource, AccountResource, ...         │  │
│  │  - JSON via Jackson + Yasson (JSON-B)         │  │
│  └───────────────────┬───────────────────────────┘  │
│                      │ CDI injection               │
│  ┌───────────────────▼───────────────────────────┐  │
│  │  backend.jar  (Business logic)                │  │
│  │  - JPA Entities (Hibernate 6.4)               │  │
│  │  - Repository pattern (DAO)                   │  │
│  │  - Business managers / services               │  │
│  └───────────────────┬───────────────────────────┘  │
│                      │ JDBC                        │
└───────────────────────┼─────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────┐
│                 MySQL 8.0                           │
└─────────────────────────────────────────────────────┘
```

**webapp** — presentation layer. JSF XHTML pages rendered server-side with PrimeFaces components. Backing beans handle UI logic and delegate to `BackendApiService` which calls the REST API. External APIs (Finnhub, CoinGecko) are consumed directly here.

**webservice** — REST API layer. Stateless JAX-RS resources expose all CRUD operations as JSON endpoints. Resources are `@RequestScoped` and inject backend services via CDI.

**backend** — core domain. JPA entities, repository interfaces with Hibernate implementations, and business managers that encapsulate domain rules (transfers, weighted average pricing, wealth aggregation, currency conversion).

---

## Tech Stack

| Category | Technology | Version |
|----------|-----------|---------|
| Language | Java | 17 LTS |
| Application Framework | Jakarta EE | 10 / 11 |
| ORM | Hibernate | 6.4 |
| Web UI | Jakarta Faces (JSF) | 11.0 |
| UI Component Library | PrimeFaces | 14.0.5 |
| REST API | Jakarta REST (JAX-RS) | 4.0 |
| Application Server | Payara Server Full | 6.2025.8 |
| Database | MySQL | 8.0 |
| Build Tool | Maven | 3.6+ |
| JSON | Jackson + Yasson (JSON-B) | 2.17.1 / 3.0.3 |
| Testing | JUnit 5 (Jupiter) | 5.10.2 |
| Containerization | Docker + Docker Compose | latest |
| Market Data — Stocks/ETFs | Finnhub API | v1 |
| Market Data — Crypto | CoinGecko API | v3 |

---

## Getting Started

### Prerequisites

- **Docker** and **Docker Compose** (to run MySQL + Payara)
- **Java 17** JDK
- **Maven 3.6+**
- A free [Finnhub API key](https://finnhub.io/) _(optional — mock data is used as fallback)_

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/marcelo-fg/e-wallet.git
cd e-wallet

# 2. (Optional) Add your Finnhub API key
#    Open docker-compose.yml and set:
#      environment:
#        - FINNHUB_API_KEY=your_key_here

# 3. Build all modules
mvn clean install -DskipTests

# 4. Start the stack
docker-compose up
```

Docker Compose starts two containers:

| Container | Image | Ports |
|-----------|-------|-------|
| `mysql` | mysql:8.0 | `3306` |
| `payara` | payara/server-full:6.2025.8-jdk17 | `8080` (HTTP), `4848` (admin) |

The WAR files are auto-deployed to Payara via volume mounts. Allow ~30 seconds for the MySQL health check to pass and Payara to finish deploying.

Once ready, open:

| Interface | URL |
|-----------|-----|
| Web application | http://localhost:8080/webapp |
| REST API base | http://localhost:8080/webservice/api |
| Payara Admin Console | http://localhost:4848 |

### Seed data

To populate the database with sample users, accounts, and portfolios:

```bash
curl -X POST http://localhost:8080/webservice/api/admin/populate
```

Sample login after seeding:

```
Email:    user1@example.com
Password: password123
```

---

## Configuration

All runtime configuration is handled through environment variables passed to Docker Compose (or set directly on the Payara JVM).

| Variable | Description | Default |
|----------|-------------|---------|
| `FINNHUB_API_KEY` | Finnhub API key for stock/ETF prices | _(none — mock data used)_ |
| `MYSQL_ROOT_PASSWORD` | MySQL root password | `root` |
| `MYSQL_DATABASE` | Database name | `ewallet_db` |

Default admin credentials:

| Service | Username | Password |
|---------|----------|----------|
| MySQL | `root` | `root` |
| Payara Admin Console | `admin` | `admin` |

> **Warning:** Change all default passwords before any kind of public deployment.

---

## REST API Reference

Base URL: `http://localhost:8080/webservice/api`

All endpoints consume and produce `application/json`.

---

### Users

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/users/register` | Register a new user |
| `POST` | `/users/login` | Authenticate (returns user object) |
| `GET` | `/users` | List all users |
| `GET` | `/users/{id}` | Get user by ID |
| `PUT` | `/users/{id}` | Update user |
| `DELETE` | `/users/{id}` | Delete user |

**Register payload:**
```json
{
  "firstName": "Alice",
  "lastName": "Dupont",
  "email": "alice@example.com",
  "password": "secret"
}
```

**Login payload:**
```json
{
  "email": "alice@example.com",
  "password": "secret"
}
```

---

### Accounts

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/accounts/user/{userId}` | Create a bank account for a user |
| `GET` | `/accounts/{id}` | Get account by ID |
| `PUT` | `/accounts/{id}` | Update account |
| `DELETE` | `/accounts/{id}` | Delete account |

**Create account payload:**
```json
{
  "name": "Main Checking",
  "balance": 1000.00,
  "currency": "USD"
}
```

---

### Transactions

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/transactions` | Create a transaction (deposit / withdraw / transfer) |
| `GET` | `/transactions/{id}` | Get transaction by ID |
| `GET` | `/transactions/account/{accountId}` | List transactions for an account |
| `GET` | `/transactions/user/{userId}` | List all transactions for a user |
| `DELETE` | `/transactions/{id}` | Delete a transaction |

**Transaction types:** `DEPOSIT`, `WITHDRAWAL`, `TRANSFER`

**Create transaction payload:**
```json
{
  "accountId": 1,
  "type": "DEPOSIT",
  "amount": 500.00,
  "description": "Monthly salary"
}
```

---

### Portfolios

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/portfolios` | Create a portfolio |
| `GET` | `/portfolios/{id}` | Get portfolio by ID |
| `DELETE` | `/portfolios/{id}` | Delete portfolio |

**Create portfolio payload:**
```json
{
  "name": "Growth Portfolio",
  "userId": 1
}
```

---

### Assets

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/portfolios/{id}/assets` | Add an asset position to a portfolio |
| `GET` | `/portfolios/{id}/assets` | List all asset positions in a portfolio |
| `DELETE` | `/portfolios/{id}/assets/{assetId}` | Remove an asset position |

**Asset types:** `STOCK`, `ETF`, `CRYPTO`

**Add asset payload:**
```json
{
  "symbol": "AAPL",
  "type": "STOCK",
  "quantity": 10,
  "averagePurchasePrice": 175.50
}
```

---

### Portfolio Transactions (Trades)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/portfolio-transactions` | Record a buy or sell trade |
| `GET` | `/portfolio-transactions/{portfolioId}` | Get trade history for a portfolio |

**Trade types:** `BUY`, `SELL`

**Record trade payload:**
```json
{
  "portfolioId": 1,
  "symbol": "AAPL",
  "type": "BUY",
  "quantity": 5,
  "price": 180.00
}
```

On a `BUY`, the asset's average cost basis is recalculated automatically. On a `SELL`, quantity is reduced and positions reaching zero are removed.

---

### Wealth

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/wealth/{userId}` | Get aggregated wealth for a user |

**Response:**
```json
{
  "totalCash": 2500.00,
  "totalEquity": 18430.50,
  "totalWealth": 20930.50,
  "currency": "USD",
  "snapshotDate": "2026-04-14"
}
```

---

### Admin

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/admin/populate` | Seed the database with sample data |

---

## Database

MySQL 8.0. The schema is created and updated automatically by Hibernate (`hbm2ddl.auto=update`). The file `database/migration_v1.sql` contains an idempotent full migration that can be re-run safely.

### Schema overview

```
users
 ├── accounts
 │    └── transactions
 └── portfolios
      ├── assets
      └── portfolio_transactions

wealth_trackers
 └── wealth_history
```

### Design conventions

| Convention | Detail |
|------------|--------|
| Monetary precision | `DECIMAL(19,4)` |
| Crypto quantity precision | `DECIMAL(19,8)` |
| Optimistic locking | `version` column on every table (`@Version`) |
| Audit columns | `created_at`, `updated_at` on key tables |
| Referential integrity | `ON DELETE CASCADE` on all foreign keys |
| IDs | Auto-increment `BIGINT` primary keys |

---

## Project Structure

```
e-wallet/
├── backend/                        # JAR — domain layer
│   └── src/main/java/.../ewallet/
│       ├── config/                 # JPA / EntityManager producers
│       ├── model/                  # JPA entities (User, Account, Portfolio, ...)
│       ├── repository/             # DAO interfaces + Hibernate implementations
│       └── service/
│           ├── CurrencyConverter.java
│           └── business/           # UserManager, AccountManager, PortfolioTransactionManager, TotalWealthService
│
├── webservice/                     # WAR — REST API layer
│   └── src/main/java/.../webservice/
│       ├── ApiConnector.java       # @ApplicationPath("/api")
│       ├── UserResource.java
│       ├── AccountResource.java
│       ├── PortfolioResource.java
│       ├── AssetResource.java
│       ├── TransactionResource.java
│       ├── PortfolioTransactionResource.java
│       ├── WealthTrackerResource.java
│       └── DataPopulationResource.java
│
├── webapp/                         # WAR — presentation layer
│   └── src/main/java/.../webapp/
│       ├── filter/AuthFilter.java  # Session-based auth guard
│       ├── ui/                     # JSF backing beans
│       │   ├── LoginBean.java
│       │   ├── RegisterBean.java
│       │   ├── DashboardBean.java
│       │   ├── AccountBean.java
│       │   ├── TransactionBean.java
│       │   ├── PortfolioBean.java
│       │   └── TotalWealthBean.java
│       └── service/
│           ├── BackendApiService.java      # REST client → webservice
│           ├── FinnhubService.java         # Stock/ETF prices
│           ├── CoinGeckoService.java       # Crypto prices
│           ├── MarketDataService.java      # Aggregates market data
│           ├── PriceCacheService.java      # In-memory price cache
│           ├── WealthCalculatorService.java
│           └── MockDataService.java        # Fallback when APIs unavailable
│   └── src/main/webapp/
│       ├── login.xhtml
│       ├── register.xhtml
│       ├── dashboard.xhtml
│       ├── accounts.xhtml
│       ├── transactions.xhtml
│       ├── portfolio.xhtml
│       ├── portfolio_analytics.xhtml
│       └── totalwealth.xhtml
│
├── database/
│   └── migration_v1.sql            # Idempotent full schema migration
├── docker-compose.yml
└── pom.xml                         # Parent POM
```

---

## Running Tests

JUnit 5 unit tests are located in `backend/src/test/`:

| Test Class | Coverage |
|------------|----------|
| `UserTest` | User creation, account and portfolio associations |
| `AccountTest` | Account operations and balance logic |
| `PortfolioTest` | Portfolio creation and asset management |
| `AssetTest` | Asset creation and pricing |
| `TransactionTest` | Transaction recording |
| `WealthTrackerTest` | Wealth aggregation calculations |

```bash
# Run all tests
mvn test

# Run a specific test class
mvn test -Dtest=PortfolioTest

# Build without tests
mvn clean install -DskipTests
```

---

## Known Limitations & Roadmap

| Area | Current state | Planned improvement |
|------|--------------|---------------------|
| Authentication | Session-based, plain password hashing | JWT tokens |
| API documentation | Manual (this README) | Swagger / OpenAPI |
| Market data cache | In-memory, single-node | Redis |
| Database connection pool | Payara default pool | HikariCP tuning |
| Currency support | USD / CHF | Pluggable via `CurrencyConverter` |
| API key management | Environment variables | Secrets manager |
| Audit logging | Timestamps only | Full event log for financial ops |
| Access control | Single user role | RBAC (admin, read-only, etc.) |
| Test coverage | Basic unit tests | Integration tests + API tests |

---

## License

MIT
