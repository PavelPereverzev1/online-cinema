# Online Cinema API

A modern, production-ready REST API for an online cinema platform built with **FastAPI**, **PostgreSQL**, **SQLAlchemy 2.0**, and
**Docker**. Features user authentication, movie management, shopping cart, orders, payments (Stripe), and an admin panel.

## 🚀 Features

### Core Functionality
- **User Management**: Registration, JWT authentication (access/refresh tokens), email verification, password reset
- **Role-Based Access Control**: User, Moderator, Admin roles with granular permissions
- **Movie Catalog**: Movies with genres, stars, directors, certifications, ratings, reviews, and favorites
- **Shopping Cart**: Add/remove movies, persistent cart per user
- **Orders & Payments**: Order creation, Stripe payment integration, webhook handling
- **Social Features**: Movie ratings (1-10), likes/dislikes, favorites, threaded comments with replies

### Technical Features
- **Async/Await**: Fully asynchronous with FastAPI and SQLAlchemy 2.0 async
- **Database Migrations**: Alembic for schema versioning
- **Background Tasks**: Celery + Redis for async processing (emails, notifications)
- **Task Monitoring**: Flower dashboard for Celery
- **Admin Panel**: SQLAdmin interface for data management
- **Object Storage**: MinIO (S3-compatible) for media files
- **Email Testing**: MailHog for development
- **API Documentation**: Auto-generated OpenAPI/Swagger UI
- **Code Quality**: Ruff linting, MyPy type checking, pre-commit hooks

## 🛠 Tech Stack

| Category | Technology |
|----------|------------|
| **Framework** | FastAPI 0.139 |
| **Language** | Python 3.13 |
| **Database** | PostgreSQL 17 (asyncpg) |
| **ORM** | SQLAlchemy 2.0 (async) |
| **Migrations** | Alembic |
| **Auth** | PyJWT (HS256), pwdlib (Argon2) |
| **Payments** | Stripe |
| **Task Queue** | Celery + Redis 7 |
| **Monitoring** | Flower |
| **Admin** | SQLAdmin |
| **Storage** | MinIO (S3-compatible) |
| **Email** | SMTP (MailHog for dev) |
| **Containerization** | Docker, Docker Compose |
| **Reverse Proxy** | Nginx |
| **Package Manager** | uv |
| **Linting** | Ruff |
| **Type Checking** | MyPy |
| **Testing** | pytest + pytest-asyncio |

## 📁 Project Structure

```
online-cinema/
├── alembic/                 # Database migrations
│   ├── migrations/          # Migration scripts
│   └── env.py               # Alembic environment
├── commands/                # Shell scripts for Docker entrypoints
│   ├── run_migration.sh     # Run Alembic migrations
│   ├── run_web_server_dev.sh
│   ├── run_web_server_prod.sh
│   ├── setup_minio.sh       # MinIO bucket setup
│   └── deploy.sh
├── configs/
│   └── nginx/               # Nginx configuration
├── docker/                  # Dockerfiles for services
│   ├── minio_mc/
│   ├── nginx/
│   └── tests/
├── docs/
│   └── development.md       # Development setup guide
├── src/
│   ├── admin/               # SQLAdmin panel setup
│   ├── api/
│   │   ├── dependencies.py  # FastAPI dependencies (auth, pagination, etc.)
│   │   ├── exceptions.py    # Custom exception handlers
│   │   └── v1/              # API v1 endpoints
│   │       ├── api.py       # Router aggregation
│   │       ├── movies.py    # Movies, genres, stars, directors, ratings, comments
│   │       ├── users.py     # Auth, profiles, avatars
│   │       ├── orders.py    # Orders, cart
│   │       └── payments.py  # Stripe integration
│   ├── core/
│   │   ├── config.py        # Pydantic settings
│   │   ├── database.py      # SQLAlchemy async engine/session
│   │   └── celery.py        # Celery app configuration
│   ├── exceptions/          # Domain-specific exceptions
│   ├── models/              # SQLAlchemy models
│   │   ├── users.py         # User, Profile, Groups
│   │   ├── movies.py        # Movie, Genre, Star, Director, Rating, Comment, Like, Favorite
│   │   ├── orders.py        # Order, OrderItem, Cart
│   │   ├── payments.py      # Payment records
│   │   └── tokens.py        # Activation, reset, refresh tokens
│   └── main.py              # FastAPI app factory
├── docker-compose.yml       # Development environment
├── docker-compose-prod.yml  # Production environment
├── docker-compose-tests.yml # Test environment
├── Dockerfile               # Multi-stage build
├── pyproject.toml           # Project config (dependencies, tools)
├── pytest.ini
├── ruff.toml
├── .env.sample              # Environment variables template
└── README.md
```

## 🏁 Quick Start (Development)

### Prerequisites
- Docker & Docker Compose
- [uv](https://docs.astral.sh/uv/) (for local development without Docker)

### 1. Clone and Configure
```bash
git clone <repository-url>
cd online-cinema
cp .env.sample .env
# Edit .env with your configuration (see Environment Variables below)
```

### 2. Start Development Stack
```bash
docker-compose up -d --build
```

This starts:
- **API** → `http://localhost:8000`
- **API Docs (Swagger)** → `http://localhost:8000/docs`
- **API Docs (ReDoc)** → `http://localhost:8000/redoc`
- **Admin Panel** → `http://localhost:8000/admin`
- **PostgreSQL** → `localhost:5432`
- **Redis** → `localhost:6379`
- **MailHog (Email UI)** → `http://localhost:8025`
- **Flower (Celery Monitor)** → `http://localhost:5555`

### 3. Run Migrations (if needed)
```bash
docker-compose exec api alembic upgrade head
```

## 🔧 Local Development (Without Docker)

### Setup
```bash
# Install uv
# Windows:
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
# macOS/Linux:
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create virtual environment
uv venv

# Install dependencies
uv sync

# Activate environment
# Windows:
.venv\Scripts\activate
# Linux/macOS:
source .venv/bin/activate
```

### Run Services Locally
You'll need PostgreSQL and Redis running locally (or via Docker):
```bash
# Start only PostgreSQL and Redis
docker-compose up -d db redis

# Run migrations
alembic upgrade head

# Start API with hot reload
uvicorn main:app --host 0.0.0.0 --port 8000 --reload

# Start Celery worker (separate terminal)
celery -A core.celery worker --loglevel=INFO

# Start Celery beat (separate terminal)
celery -A core.celery beat --loglevel=INFO
```

## 🧪 Testing

```bash
# Run all tests
docker-compose -f docker-compose-tests.yml up --build --abort-on-container-exit

# Or locally with pytest
uv run pytest

# With coverage
uv run pytest --cov=src --cov-report=html
```

## 📚 API Endpoints

Base URL: `http://localhost:8000/api/v1`

### Authentication (`/users`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/users/register` | Register new user |
| POST | `/users/login` | Login (returns access + refresh tokens) |
| POST | `/users/refresh` | Refresh access token |
| POST | `/users/logout` | Logout (revoke refresh token) |
| GET | `/users/me` | Get current user profile |
| PATCH | `/users/me` | Update profile |
| POST | `/users/activate` | Activate account via email token |
| POST | `/users/forgot-password` | Request password reset |
| POST | `/users/reset-password` | Reset password with token |

### Movies (`/movies`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/movies` | List movies (pagination, filters) |
| GET | `/movies/{movie_id}` | Get movie details |
| POST | `/movies` | Create movie (admin) |
| PATCH | `/movies/{movie_id}` | Update movie (admin) |
| DELETE | `/movies/{movie_id}` | Delete movie (admin) |
| GET | `/movies/{movie_id}/comments` | Get movie comments |
| POST | `/movies/{movie_id}/comments` | Add comment |
| POST | `/movies/{movie_id}/rating` | Rate movie (1-10) |
| POST | `/movies/{movie_id}/like` | Like/dislike movie |
| POST | `/movies/{movie_id}/favorite` | Add/remove favorite |

### Genres/Stars/Directors/Certifications
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/genres` | List genres |
| POST | `/genres` | Create genre (admin) |
| GET | `/stars` | List stars |
| POST | `/stars` | Create star (admin) |
| GET | `/directors` | List directors |
| POST | `/directors` | Create director (admin) |
| GET | `/certifications` | List certifications |
| POST | `/certifications` | Create certification (admin) |

### Cart & Orders (`/orders`, `/cart`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/cart` | Get user's cart |
| POST | `/cart/items` | Add item to cart |
| DELETE | `/cart/items/{movie_id}` | Remove item from cart |
| POST | `/orders` | Create order from cart |
| GET | `/orders` | List user's orders |
| GET | `/orders/{order_id}` | Get order details |

### Payments (`/payments`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/payments/create-checkout-session` | Create Stripe checkout session |
| POST | `/payments/webhook` | Stripe webhook handler |

## ⚙️ Environment Variables

Copy `.env.sample` to `.env` and configure:

| Variable | Description | Default |
|----------|-------------|---------|
| **Database** | | |
| `POSTGRES_DB` | Database name | `movies_db` |
| `POSTGRES_USER` | Database user | `postgres` |
| `POSTGRES_PASSWORD` | Database password | `some_password` |
| `POSTGRES_HOST` | Database host | `localhost` |
| `POSTGRES_DB_PORT` | Database port | `5432` |
| **Auth** | | |
| `TOKEN_SECRET_KEY` | JWT signing key (change in prod!) | `UuNi2QtnGzRdGIJmsRURVuQFthrwsr1EZu8fOomLQTZ` |
| `TOKEN_ALGORITHM` | JWT algorithm | `HS256` |
| `ACCESS_TOKEN_EXPIRE` | Access token TTL | `1 day` |
| `REFRESH_TOKEN_EXPIRE` | Refresh token TTL | `30 days` |
| **Stripe** | | |
| `STRIPE_SECRET_KEY` | Stripe secret key | `sk_test_...` |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook signing secret | `whsec_...` |
| `BASE_URL` | Public API base URL | `http://localhost:8000` |
| **Email (SMTP)** | | |
| `SMTP_HOST` | SMTP server | `mailhog` |
| `SMTP_PORT` | SMTP port | `1025` |
| `SMTP_USER` | SMTP username | - |
| `SMTP_PASSWORD` | SMTP password | - |
| `SMTP_USE_TLS` | Use TLS | `false` |
| `EMAIL_FROM` | Sender email | `no-reply@online-cinema.com` |
| **MinIO** | | |
| `MINIO_ROOT_USER` | MinIO access key | `minioadmin` |
| `MINIO_ROOT_PASSWORD` | MinIO secret key | `some_password` |
| `MINIO_HOST` | MinIO host | `minio-theater` |
| `MINIO_PORT` | MinIO port | `9000` |
| `MINIO_STORAGE` | Bucket name | `theater-storage` |
| **Celery** | | |
| `CELERY_BROKER_URL` | Redis broker URL | `redis://redis:6379/0` |
| `CELERY_RESULT_BACKEND` | Redis result backend | `redis://redis:6379/0` |

## 🏭 Production Deployment

### 1. Prepare Environment
```bash
cp .env.sample .env.prod
# Edit .env.prod with production values (especially secrets!)
```

### 2. Deploy
```bash
docker-compose -f docker-compose-prod.yml --env-file .env.prod up -d --build
```

Production stack includes:
- **Nginx** reverse proxy (port 80)
- **PostgreSQL** with persistent volume
- **pgAdmin** (port 3333, localhost only)
- **MinIO** object storage (ports 9000, 9001)
- **MailHog** for email testing
- **Redis** for Celery
- **Celery worker + beat** for background tasks
- **Flower** monitoring (port 5555)

### 3. Run Migrations
```bash
docker-compose -f docker-compose-prod.yml exec migrator /commands/run_migration.sh
```

## 🛡 Admin Panel

Access at `http://localhost:8000/admin` (development) or your domain `/admin` (production).

Default admin credentials (set via migration):
- Email: `admin@online-cinema.com`
- Password: Set via `PGADMIN_DEFAULT_PASSWORD` in `.env`

Features:
- User management (view, edit, delete, change roles)
- Movie/Genre/Star/Director/Certification CRUD
- Order & Payment monitoring
- Comment moderation

## 🔐 Security Notes

- **Change all default secrets** in production (`TOKEN_SECRET_KEY`, `POSTGRES_PASSWORD`, `STRIPE_SECRET_KEY`, etc.)
- Use **HTTPS** in production (configure SSL in Nginx)
- Set `SMTP_USE_TLS=true` for production email
- Restrict admin panel access (IP allowlist, VPN, or auth proxy)
- Rotate JWT secrets periodically
- Use strong passwords for all services

## 📦 Useful Commands

### Database
```bash
# Create new migration
alembic revision --autogenerate -m "description"

# Apply migrations
alembic upgrade head

# Rollback one migration
alembic downgrade -1

# Show migration history
alembic history
```

### Development
```bash
# Lint code
uv run ruff check .

# Format code
uv run ruff format .

# Type check
uv run mypy src

# Run pre-commit hooks
pre-commit run --all-files
```

### Docker
```bash
# View logs
docker-compose logs -f api

# Restart service
docker-compose restart api

# Rebuild and restart
docker-compose up -d --build api

# Clean up
docker-compose down -v  # removes volumes!
```

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests and linting
5. Submit a pull request

---

**Built with ❤️ using FastAPI, PostgreSQL, and Docker**