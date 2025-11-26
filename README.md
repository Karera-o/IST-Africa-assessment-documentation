# ProcureFlow - IST Africa

A modern Procure-to-Pay system for efficient procurement management with intelligent automation and seamless workflows.

## Repository Structure

This repository contains the orchestration files for the full-stack application.

- **Backend Repository**: [https://github.com/Karera-o/IST-Africa-Assessment.git](https://github.com/Karera-o/IST-Africa-Assessment.git)
- **Frontend Repository**: [https://github.com/Karera-o/IST-Africa-Assessment-Frontend.git](https://github.com/Karera-o/IST-Africa-Assessment-Frontend.git)

> **Note**: The backend and frontend are maintained as separate repositories. Clone them separately or use the docker-compose setup below.

## Live Instance

- **Frontend**: https://ist.africa.olivierkarera.com/
- **Backend API**: https://ist.backend.kareralabs.com/
- **API Documentation (OpenAPI/ReDoc)**: https://ist.backend.kareralabs.com/redoc
- **Swagger UI**: https://ist.backend.kareralabs.com/swagger/

## Quick Start

### Prerequisites

- Docker and Docker Compose installed
- Git

### Setup & Run

#### Option 1: Using Docker Compose (Recommended)

```bash
# Clone all three repositories
git clone <parent-repo-url> .
git clone <backend-repo-url> ist_backend_assessment
git clone <frontend-repo-url> ist_frontend

# Build and start all services
docker-compose up --build
```

#### Option 2: Individual Setup

See the individual repository READMEs for backend and frontend setup instructions.

This will start:

- **Backend** (Django + Gunicorn): http://localhost:8000
- **Frontend** (Next.js): http://localhost:3000
- **PostgreSQL**: localhost:5432
- **Redis**: localhost:6379
- **Celery Worker**: Running in background

### Stop Services

```bash
docker-compose down
```

### View Logs

```bash
docker-compose logs -f
```

## Environment Variables

Create a `.env` file in the root directory with the following variables:

### Required for Backend

```env
# Database
DATABASE_URL=postgresql://user:password@db:5432/dbname

# Django
SECRET_KEY=your-secret-key
DEBUG=False
ALLOWED_HOSTS=localhost,127.0.0.1,your-domain.com

# JWT Authentication (RS256)
JWT_PRIVATE_KEY=-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----
JWT_PUBLIC_KEY=-----BEGIN PUBLIC KEY-----
...
-----END PUBLIC KEY-----

# Postmark Email Service
POSTMARK_SERVER_TOKEN=your-postmark-server-token
POSTMARK_FROM_EMAIL=noreply@yourdomain.com
POSTMARK_FROM_NAME=ProcureFlow

# OpenAI/OpenRouter (for document extraction)
OPENROUTER_API_KEY=your-openrouter-api-key
# Default model: z-ai/glm-4.6

# Cloudflare R2 (for file storage)
CLOUDFLARE_R2_ACCOUNT_ID=your-account-id
CLOUDFLARE_R2_ACCESS_KEY_ID=your-access-key
CLOUDFLARE_R2_SECRET_ACCESS_KEY=your-secret-key
CLOUDFLARE_R2_BUCKET=your-bucket-name
R2_PUBLIC_BASE_URL=https://your-domain.com

# Redis
REDIS_URL=redis://redis:6379/0

# Celery
CELERY_BROKER_URL=redis://redis:6379/0
CELERY_RESULT_BACKEND=redis://redis:6379/0
```

### Optional for Frontend

```env
# Frontend environment variable (if running separately)
NEXT_PUBLIC_API_URL=http://localhost:8000/api
```

**Note**: By default, the frontend uses `http://localhost:8000/api` as the backend URL. You can override this by setting `NEXT_PUBLIC_API_URL` environment variable.

## User Roles & Permissions

### Staff

- Create purchase requests
- View and update their own pending requests
- Upload receipts for approved requests
- View their own request history

### Approvers (Level 1 & Level 2)

- View all pending requests
- Approve/reject requests at their level
- View all reviewed requests
- Access approval history

### Finance Team

- Access and interact with all requests via web UI
- Upload files
- View all purchase requests and receipts

### Admin

- Manage users and roles
- View all requests and approvals

**Default Admin Credentials:**

- Email: `admin@gmail.com`
- Password: `IST@Africa`

## API Documentation

### OpenAPI/Swagger

- **ReDoc**: https://ist.backend.kareralabs.com/redoc
- **Swagger UI**: https://ist.backend.kareralabs.com/swagger/

### Authentication

The API uses JWT (JSON Web Tokens) with RS256 algorithm. Include the access token in the Authorization header:

```
Authorization: Bearer <access_token>
```

### Main Endpoints

- `POST /api/auth/register/` - User registration
- `POST /api/auth/login/` - User login
- `GET /api/purchase-requests/` - List purchase requests
- `POST /api/purchase-requests/` - Create purchase request
- `GET /api/purchase-requests/{id}/` - Get request details
- `PUT /api/purchase-requests/{id}/` - Update request (pending only)
- `PATCH /api/purchase-requests/{id}/approve/` - Approve request
- `PATCH /api/purchase-requests/{id}/reject/` - Reject request
- `POST /api/purchase-requests/{id}/upload-receipt/` - Upload receipt
- `GET /api/purchase-requests/receipts/` - List receipts
- `GET /api/purchase-requests/dashboard/` - Dashboard statistics
- `GET /api/purchase-requests/?approval=true` - Pending approvals

## Architecture

### Backend Stack

- **Framework**: Django 5.0 + Django REST Framework
- **Database**: PostgreSQL 17
- **Cache/Queue**: Redis 7
- **Task Queue**: Celery
- **WSGI Server**: Gunicorn (6 workers)
- **Authentication**: JWT (RS256)
- **File Storage**: Cloudflare R2 (with local fallback)
- **Email**: Postmark
- **Document Processing**: pdfplumber, Tesseract OCR, OpenAI/OpenRouter

### Frontend Stack

- **Framework**: Next.js 14
- **Deployment**: Vercel
- **Styling**: Tailwind CSS

### Project Structure

```
project/
├── ist_backend_assessment/     # Django backend
│   ├── config/                 # Django settings
│   ├── user_auth/              # Authentication app
│   ├── purchase_request/       # Purchase request app
│   │   ├── models/             # Data models
│   │   ├── serializers/        # DRF serializers
│   │   ├── repositories/       # Database operations
│   │   ├── services/           # Business logic
│   │   └── views/              # API views
│   ├── base/                   # Shared utilities
│   └── utils/                  # Common utilities
├── ist_frontend/               # Next.js frontend
└── docker-compose.yml          # Docker orchestration
```

## Development

### Running Locally (without Docker)

#### Backend

```bash
cd ist_backend_assessment
uv venv
source .venv/bin/activate  # or `.venv\Scripts\activate` on Windows
uv sync
python manage.py migrate
python manage.py runserver
```

#### Frontend

```bash
cd ist_frontend
npm install
npm run dev
```

### Running Celery Worker

```bash
cd ist_backend_assessment
celery -A config worker --loglevel=info
```

## Docker Services

- **backend**: Django application with Gunicorn (6 workers)
- **frontend**: Next.js application
- **db**: PostgreSQL database
- **redis**: Redis cache and message broker
- **celery**: Celery worker for async tasks

## Security

- JWT tokens with RS256 algorithm
- Role-based access control (RBAC)
- Non-root user execution in containers
- Environment variables for sensitive data
- CORS configured for allowed origins

## Features

- User registration and authentication
- Purchase request creation with proforma upload
- Automatic document extraction (OCR + AI)
- Multi-level approval workflow
- Purchase Order generation (PDF)
- Receipt upload and validation
- Dashboard with statistics
- Email notifications
- File storage (R2 or local)

## Troubleshooting

### Database Connection Issues

Ensure PostgreSQL is running and `DATABASE_URL` is correctly set in `.env`.

### Email Not Sending

Verify `POSTMARK_SERVER_TOKEN` and `POSTMARK_FROM_EMAIL` are set correctly.

### File Upload Issues

Check Cloudflare R2 credentials or ensure local storage is properly configured.

### Celery Tasks Not Running

Ensure Redis is running and `CELERY_BROKER_URL` is correctly configured.

## 📄 License

IST Africa Technical Assessment

## Contact

For issues or questions, please contact olivierkarera2020@gmail.com .
