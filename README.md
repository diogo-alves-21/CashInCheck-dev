## CashInCheck

CashInCheck is a Ruby on Rails application for personal and shared money management.
It helps users organise their **groups, wallets, budgets, goals and transactions**, and integrates with external providers to keep balances and movements in sync.

The repository captures both the product features and the engineering practices used in the team (testing, code style, documentation and deployment tooling).

---

## Table of Contents

- [Overview](#overview)
- [Main Features](#main-features)
- [Tech Stack](#tech-stack)
- [Architecture & Domain Concepts](#architecture--domain-concepts)
- [Getting Started](#getting-started)
  - [Requirements](#requirements)
  - [Local Setup](#local-setup)
  - [Running the App](#running-the-app)
  - [Docker](#docker)
- [Quality & Tooling](#quality--tooling)
- [Documentation](#documentation)
- [Development Workflow](#development-workflow)
- [Testing](#testing)

---

## Overview

CashInCheck is a budgeting and cash‑flow control system focused on:

- **Centralising finances** across multiple wallets and groups
- **Tracking transactions and categories** with tags and budgets
- **Defining saving goals** and monitoring progress over time
- **Supporting bank and payment integrations** so information stays up to date

The app exposes:

- A **user area** where authenticated users manage their groups, wallets, budgets, goals and transactions
- An **admin area** for managing admins, users and consents
- A set of **background jobs** for synchronising data and processing external callbacks

---

## Main Features

- **Authentication and access control**
  - User and admin authentication with `devise` and `devise_invitable`
  - Role‑based access and policies with `pundit`
  - Consent management and cookie control

- **Groups, wallets and transactions**
  - Groups with members and shared wallets
  - Wallets and goals backed by money‑aware models (`money`, `money-rails`, `monetize`)
  - Transactions, tags and categories for detailed tracking
  - Budget entities to keep spending under control

- **Open banking & external integrations**
  - Bank account aggregation and transaction sync via `nordigen-ruby` (open banking)
  - Background jobs (Sidekiq + `sidekiq-scheduler`) to periodically fetch and reconcile data
  - Support for payment providers such as **EasyPay** and **PayPal** (see docs in `docs/`)

- **Admin & monitoring**
  - Admin dashboard for admins, users and consents
  - Sidekiq Web UI mounted at `/sidekiq` for job monitoring
  - Error tracking with `sentry-ruby`, `sentry-rails` and `sentry-sidekiq`

- **Modern front‑end**
  - Rails 8 view stack using **Hotwire**: `turbo-rails` and `stimulus-rails`
  - SCSS structure with reusable components, layouts and a styleguide
  - Charting via `chartkick`

---

## Tech Stack

- **Backend**
  - Ruby on Rails `~> 8.0.1`
  - PostgreSQL
  - Sidekiq + `sidekiq-scheduler` for background processing
  - Redis (for caching, queues and Action Cable)

- **Frontend**
  - Hotwire (`turbo-rails`, `stimulus-rails`)
  - SCSS via `cssbundling-rails`
  - JavaScript bundling via `jsbundling-rails`

- **DevOps / Operations**
  - `kamal` for container‑based deployments
  - `thruster` for HTTP performance on Puma
  - Docker configuration for local containers

- **Testing & Quality**
  - RSpec, Capybara, Selenium, FactoryBot
  - RuboCop, ERB Lint, ESLint, Stylelint
  - Brakeman, Bundler‑Audit

---

## Architecture & Domain Concepts

The codebase follows a **conventional Rails structure**, documented in more detail in `docs/code.md`.
Some core domain concepts:

- **User / Admin**: authenticated actors of the system
- **Group** and **Member**: collaborative space where multiple users share wallets and budgets
- **Wallet / Goal**: money containers with balances, targets and open‑banking links
- **Transaction**, **Category**, **Tag**: structures for classifying and analysing movements
- **Budget / BudgetUnit**: entities for planning and enforcing spending limits
- **Consent / AcceptedConsent**: records tied to user acceptance of required terms

The app uses:

- **Service objects** (`app/services/`) to encapsulate complex logic and external API calls
- **Jobs** (`app/jobs/`) for asynchronous work such as transaction sync and payment notifications
- **Policies** (`app/policies/`) for authorisation rules
- **Validators & concerns** to keep models small and focused

For a visual overview, see the ER diagram in `diagram_er.puml` and `docs/er_diagram.md`.

---

## Getting Started

### Requirements

Before running the project locally you will need:

- **Ruby** (version defined in `Gemfile`; using a version manager like `asdf` is recommended)
- **Bundler**
- **Yarn**
- **PostgreSQL**
- **Redis**

Check `docs/setup.md` for more details and platform‑specific hints.

### Local Setup

1. **Clone the repository**

   ```bash
   git clone git@github.com:your-org/cash_in_check.git
   cd cash_in_check
   ```

2. **Configure environment variables & secrets**

   - Create `config/local_env.yml` and copy the contents of `config/local_env.example.yml`
   - Adjust credentials and API keys to your local environment
   - Set the **Nordigen secrets**:
     - `NORDIGEN_SECRET_ID`
     - `NORDIGEN_SECRET_KEY`
   - You can define these either in your shell, in a local `.env` file, or via your preferred secrets manager, as long as they are available as environment variables when Rails boots.

3. **Install dependencies and set up the database**

   ```bash
   bin/setup
   ```

### Running the App

Start the application with:

```bash
bin/dev
```

Then open `http://localhost:3000` in your browser.

### Docker

Docker support is available but still considered **beta**.
The basic flow is:

- Build the image:

  ```bash
  docker build .
  ```

- Create and prepare the database:

  ```bash
  docker-compose run --rm web rails db:create
  docker-compose run --rm web rails db:migrate
  docker-compose run --rm web rails db:seed
  ```

- Start the stack:

  ```bash
  docker-compose up
  ```

Then open `http://0.0.0.0:3000`.

For more details, see the **Docker** section in `docs/setup.md`.

---

## Quality & Tooling

The project uses multiple tools to enforce consistency and quality (see `docs/code.md`):

- **Ruby**: RuboCop
- **Views**: ERB Lint
- **JavaScript**: ESLint
- **Sass**: Stylelint
- **Security & dependencies**: Brakeman, Bundler‑Audit

Typical commands:

```bash
bundle exec rubocop
bundle exec erblint --lint-all
yarn eslint
yarn stylelint
bundle exec brakeman
bundle exec bundler-audit
```

---

## Documentation

Additional documentation lives under the `docs/` directory:

- `docs/setup.md` – detailed local setup, Docker usage and editor configuration
- `docs/code.md` – file structure, model organisation, git workflow and conventions
- `docs/easypay.md` – EasyPay integration, configuration and payment flows
- `docs/paypal.md` – PayPal integration and sandbox setup
- `docs/receive_emails.md`, `docs/tunneling.md`, `docs/styleguide.md`, etc. – auxiliary topics

The codebase also uses **YARD** for in‑code documentation:

```bash
yard server -r
```

Then open `http://localhost:8808` to browse the generated docs.

---

## Testing

The test suite is based on:

- **RSpec** for unit and integration tests
- **Capybara + Selenium + webdrivers** for system/browser tests
- **FactoryBot** and **Faker** for factories and sample data
- **DatabaseCleaner**, **Timecop** and `formulaic` for test helpers

Run the full test suite with:

```bash
bundle exec rspec
```
