# E-Wallet — Personal Finance Management Platform

A full-stack **Jakarta EE** application for managing bank accounts, investment portfolios, and personal wealth tracking with real-time market data integration.

![Java](https://img.shields.io/badge/Java-17-007396?style=flat&logo=java)
![Jakarta EE](https://img.shields.io/badge/Jakarta%20EE-10%2F11-f89820?style=flat)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=flat&logo=mysql&logoColor=white)
![Payara](https://img.shields.io/badge/Payara-6.2025-0b6cb7?style=flat)
![PrimeFaces](https://img.shields.io/badge/PrimeFaces-14.0-e97f1b?style=flat)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker&logoColor=white)
![Maven](https://img.shields.io/badge/Maven-3-C71A36?style=flat&logo=apachemaven&logoColor=white)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [API Reference](#api-reference)
- [Project Structure](#project-structure)
- [Database Schema](#database-schema)

---

## Overview

E-Wallet is an enterprise-grade personal finance application built with a clean **3-tier architecture**: a JSF/PrimeFaces web UI, a JAX-RS REST API layer, and a JPA/Hibernate backend. It allows users to:

- Manage multiple bank accounts and track transactions
- Build and monitor investment portfolios (stocks, ETFs, crypto)
- Visualize total wealth with historical trends and analytics
- Get real-time market prices from **Finnhub** and **CoinGecko**

---

## Features

### Bank Accounts
- Create, view, update and delete accounts
- Deposit, withdraw and transfer between accounts
- Full transaction history with audit trail

### Investment Portfolios
- Multiple portfolios per user
- Supports **stocks**, **ETFs** and **cryptocurrencies**
- Weighted average pricing on buy/sell operations
- Automatic cleanup of zero-quantity positions

### Wealth Dashboard
- Aggregated view of cash + crypto + stocks
- Historical wealth tracking with growth rate
- USD / CHF currency conversion
- Interactive charts (PrimeFaces)

### Market Data
- Real-time stock and ETF prices via [Finnhub API](https://finnhub.io/)
- Real-time crypto prices via [CoinGecko API](https://www.coingecko.com/)
- In-memory price caching layer for performance

### Security
- Password hashing and validation following **OWASP 2024** and **NIST SP800-63B** guidelines
- Session-based authentication with `AuthFilter`
- Optimistic locking on all entities (`@Version`) to prevent race conditions

---

## Architecture


┌──────────────────────────────────────────┐
│ Webapp (JSF + PrimeFaces) │ :8080
│ Login · Dashboard · Accounts · │
│ Portfolios · Transactions · Analytics │
└──────────────────┬───────────────────────┘
│ REST (HTTP/JSON)
▼
┌──────────────────────────────────────────┐
│ WebService (JAX-RS REST API) │ /api/*
│ /users · /accounts · /portfolios │
│ /assets · /transactions · /wealth │
└──────────────────┬───────────────────────┘
│ JPA / Hibernate
▼
┌──────────────────────────────────────────┐
│ Backend (Business Logic) │
│ Entities · Repositories · Services │
│ Managers · Currency Converter │
└──────────────────┬───────────────────────┘
│ JDBC
▼
┌──────────────────────────────────────────┐
│ MySQL 8.0 (ewallet_db) │ :3306
└──────────────────────────────────────────┘


The project is structured as a **multi-module Maven project**:

| Module       | Packaging | Role                                |
|--------------|-----------|-------------------------------------|
| `backend`    | JAR       | Entities, repositories, services    |
| `webservice` | WAR       | REST API (JAX-RS, deployed on Payara) |
| `webapp`     | WAR       | Web UI (JSF, PrimeFaces, CDI beans) |

---

## Tech Stack

| Layer          | Technology                                  |
|----------------|---------------------------------------------|
| Language       | Java 17                                     |
| Framework      | Jakarta EE 10 / 11 (CDI, JPA, JAX-RS, JSF) |
| ORM            | Hibernate 6.4 (JPA 3.1)                    |
| UI Components  | PrimeFaces 14.0                             |
| Application Server | Payara Server 6 (GlassFish fork)      |
| Database       | MySQL 8.0                                   |
| Build Tool     | Maven 3                                     |
| Infrastructure | Docker + Docker Compose                     |
| External APIs  | Finnhub (stocks/ETFs), CoinGecko (crypto)  |
| Serialization  | Jackson + Yasson (JSON-B)                  |
| Testing        | JUnit 5 (Jupiter)                          |

---

## Getting Started

### Prerequisites

- [Docker](https://www.docker.com/) and Docker Compose
- [Maven 3](https://maven.apache.org/) and Java 17 JDK
- A [Finnhub](https://finnhub.io/) API key (free tier is sufficient)

### 1. Clone the repository

```bash
git clone https://github.com/marcelo-fg/e-wallet.git
cd e-wallet

2. Configure environment variables
Open docker-compose.yml and set your Finnhub API key:

environment:
  - FINNHUB_API_KEY=your_api_key_here

3. Build the project
mvn clean install -DskipTests

4. Start the environment
docker-compose up

This will start:

MySQL on port 3306 (database: ewallet_db, credentials: root / root)
Payara Server on port 8080 (HTTP) and 4848 (admin console)
The WAR files are automatically deployed via the Payara autodeploy/ mechanism.

5. Access the application
Service	URL
Web App	http://localhost:8080/webapp
REST API	http://localhost:8080/webservice/api
Payara Admin	http://localhost:4848
6. Initialize sample data (optional)
curl -X POST http://localhost:8080/webservice/api/populate

API Reference
Base URL: http://localhost:8080/webservice/api

Users
Method	Endpoint	Description
POST	/users/register	Register a new user
POST	/users/login	Authenticate a user
GET	/users/{id}	Get user by ID
GET	/users	List all users
Accounts
Method	Endpoint	Description
POST	/accounts/user/{userId}	Create an account for a user
GET	/accounts/{id}	Get account details
DELETE	/accounts/{id}	Delete an account
Portfolios
Method	Endpoint	Description
POST	/portfolios	Create a portfolio
GET	/portfolios/{id}	Get portfolio details
DELETE	/portfolios/{id}	Delete a portfolio
Assets
Method	Endpoint	Description
GET	/portfolios/{id}/assets	List assets in portfolio
POST	/portfolios/{id}/assets	Add an asset
DELETE	/portfolios/{id}/assets/{assetId}	Remove an asset
Portfolio Transactions
Method	Endpoint	Description
GET	/portfolios/{id}/portfolio-transactions	Get trade history
POST	/portfolios/{id}/portfolio-transactions	Record a buy/sell
Wealth Tracker
Method	Endpoint	Description
GET	/wealth	Get current aggregated wealth
GET	/wealth/history	Get historical wealth values
Project Structure
e-wallet/
├── backend/
│   └── src/main/java/org/groupm/ewallet/
│       ├── model/              # JPA Entities (User, Account, Portfolio, Asset, …)
│       ├── repository/         # JPA Repositories (data access layer)
│       └── service/
│           ├── business/       # Business managers (UserManager, AccountManager, …)
│           └── external/       # Currency converter
├── webservice/
│   └── src/main/java/org/groupm/ewallet/webservice/
│       ├── *Resource.java      # JAX-RS REST resources
│       └── ApiConnector.java   # External API client utility
├── webapp/
│   └── src/main/java/org/groupm/ewallet/webapp/
│       ├── ui/                 # JSF backing beans (LoginBean, DashboardBean, …)
│       ├── service/            # App services (FinnhubService, CoinGeckoService, …)
│       ├── model/              # Local DTOs
│       └── filter/             # AuthFilter (session protection)
│   └── src/main/webapp/        # XHTML pages (JSF views)
├── database/
│   └── migration_v1.sql        # Complete schema with indexes & FK constraints
├── docker-compose.yml
└── pom.xml                     # Parent Maven POM

Database Schema
The application uses 9 tables with consistent conventions:

DECIMAL(19,4) for monetary amounts, DECIMAL(19,8) for crypto quantities
version column on every table for optimistic locking
created_at / updated_at audit timestamps on key tables
ON DELETE CASCADE foreign keys for referential integrity
users ──< accounts ──< transactions
  │
  └──< portfolios ──< assets
         │        └──< portfolio_transactions
         │
  └──── wealth_trackers ──< wealth_history
