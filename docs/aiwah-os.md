# AIWAH OS – Technical Documentation

---

## Overview

AIWAH OS is a self-hosted infrastructure boilerplate built with Docker Compose services. It provides a backend stack with authentication, database, real-time capabilities, and workflow automation using official Supabase self-hosting and N8N workflow engine. Currently, each service runs independently and requires manual coordination.

---

## Architecture

### Current Service Architecture

The system consists of 3 independent services, each with their own docker-compose.yml:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CURRENT ARCHITECTURE                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐        │
│  │   UI Service    │    │   Supabase       │    │   N8N Service   │        │
│  │   Port: 3000    │    │   Service        │    │   Port: 5678    │        │
│  │                 │    │   Port: 8000     │    │                 │        │
│  │ ┌─────────────┐ │    │ ┌──────────────┐ │    │ ┌─────────────┐ │        │
│  │ │  Next.js    │ │    │ │   Studio     │ │    │ │    N8N      │ │        │
│  │ │  Container  │ │    │ │   Kong       │ │    │ │  Workflow   │ │        │
│  │ │             │ │    │ │   Auth       │ │    │ │   Engine    │ │        │
│  │ └─────────────┘ │    │ │   REST       │ │    │ └─────────────┘ │        │
│  │                 │    │ │   Realtime   │ │    │ ┌─────────────┐ │        │
│  │ Independent     │    │ │   Storage    │ │    │ │ Dedicated   │ │        │
│  │ Service         │    │ │   Database   │ │    │ │ PostgreSQL  │ │        │
│  └─────────────────┘    │ │   etc...     │ │    │ └─────────────┘ │        │
│                          │ └──────────────┘ │    │                 │        │
│                          │                  │    │ Independent     │        │
│                          │ Independent      │    │ Service         │        │
│                          │ Service Stack    │    └─────────────────┘        │
│                          └──────────────────┘                               │
│                                                                             │
│                    Manual coordination required                             │
│                    No shared networks currently                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Technology Stack

**Infrastructure:**
- Docker Compose (3 independent service stacks)
- Official Supabase self-hosting (latest stable)
- Manual service coordination
- Individual service management

**Application:**
- Next.js 13+ with TypeScript and Tailwind CSS
- Official Supabase client libraries
- Kong API Gateway for routing and security

**Backend:**
- Official Supabase PostgreSQL with all extensions
- PostgREST auto-generated REST API
- GoTrue JWT authentication
- Supabase Realtime WebSocket
- Supabase Storage with local file backend

**Automation:**
- N8N workflow engine with dedicated PostgreSQL
- Complete isolation from user data
- API-based integration capability with Supabase

---

## Service Breakdown

### 1. UI Service (`services/ui/`)

**Structure:**
```
services/ui/
├── docker-compose.yml    # UI service definition
├── Dockerfile           # Multi-stage Next.js build
├── package.json         # Dependencies
├── src/                 # Source code
├── public/              # Static assets
└── .gitignore           # UI-specific ignore patterns
```

**Configuration:**
- **Container:** `aiwah-ui`
- **Port:** 3000
- **Environment:** Supabase URL and API keys
- **Build:** Custom Dockerfile with Next.js optimization

**Docker Compose Configuration:**
```yaml
services:
  ui:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: aiwah-ui
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_SUPABASE_URL=${SUPABASE_URL:-http://localhost:8000}
      - NEXT_PUBLIC_SUPABASE_ANON_KEY=${ANON_KEY}
    restart: unless-stopped
```

### 2. Supabase Service (`services/supabase/`)

**Structure:**
```
services/supabase/
├── docker-compose.yml    # Official Supabase stack
├── .env                 # Environment configuration
├── .gitignore           # Supabase-specific ignore patterns
├── reset.sh             # Reset utility script
└── volumes/             # Configuration and data
    ├── api/             # Kong configuration
    ├── db/              # Database initialization
    │   ├── init/        # Initialization scripts (tracked)
    │   └── *.sql        # Schema files (tracked)
    ├── storage/         # File storage
    ├── functions/       # Edge functions
    ├── pooler/          # Connection pooler config
    └── logs/            # Log configuration
```

**Services Included:**
- **studio:** Supabase Studio (database admin)
- **kong:** API Gateway and router
- **auth:** GoTrue authentication service
- **rest:** PostgREST API generator
- **realtime:** WebSocket service
- **storage:** File storage service
- **imgproxy:** Image transformation
- **meta:** Database metadata service
- **functions:** Edge functions runtime
- **analytics:** Logflare analytics
- **db:** PostgreSQL database
- **vector:** Log collection
- **supavisor:** Connection pooler

**Important Git Tracking:**
- ✅ **Tracked:** `volumes/db/*.sql` (schema files)
- ✅ **Tracked:** `volumes/db/init/` (initialization scripts)
- ❌ **Ignored:** `volumes/db/data/` (runtime database data)
- ❌ **Ignores:** User-generated dumps

### 3. N8N Service (`services/n8n/`)

**Structure:**
```
services/n8n/
├── docker-compose.yml    # N8N + dedicated PostgreSQL
└── .gitignore           # N8N-specific ignore patterns
```

**Services:**
- **n8n:** Workflow engine container
- **postgres:** Dedicated PostgreSQL database

**Docker Compose Configuration:**
```yaml
services:
  postgres:
    image: postgres:16
    environment:
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=${N8N_DB_PASSWORD}
      - POSTGRES_DB=n8n
    volumes:
      - n8n_postgres_data:/var/lib/postgresql/data

  n8n:
    image: docker.n8n.io/n8nio/n8n
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_USERNAME}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASSWORD}
    ports:
      - 5678:5678
    volumes:
      - n8n_data:/home/node/.n8n
```

**Isolation:**
- Completely separate from Supabase user data
- Own PostgreSQL instance for workflow data
- Can connect to Supabase via HTTP APIs

---

## Current Deployment Process

### Manual Service Management

Each service must be managed independently:

**1. Start Supabase (Required First):**
```bash
cd services/supabase
docker-compose up -d
```

**2. Start N8N:**
```bash
cd services/n8n
docker-compose up -d
```

**3. Start UI:**
```bash
cd services/ui
docker-compose up -d
```

### Service Commands

**Per-Service Management:**
```bash
# Supabase
cd services/supabase
docker-compose up -d        # Start
docker-compose down         # Stop
docker-compose down -v      # Stop and remove volumes
docker-compose logs -f      # View logs
docker-compose ps           # Check status

# N8N
cd services/n8n
docker-compose up -d
docker-compose down
docker-compose logs -f

# UI
cd services/ui
docker-compose up -d
docker-compose down
docker-compose logs -f
```

### Environment Configuration

**Supabase Configuration (`services/supabase/.env`):**
```bash
# Database
POSTGRES_PASSWORD=your-secure-password

# JWT Configuration
JWT_SECRET=your-jwt-secret-key
ANON_KEY=your-anon-key
SERVICE_ROLE_KEY=your-service-role-key

# Admin Access
DASHBOARD_USERNAME=admin
DASHBOARD_PASSWORD=your-admin-password

# URLs
SITE_URL=http://localhost:3000
```

**N8N Configuration (Environment Variables):**
```bash
N8N_USERNAME=admin
N8N_PASSWORD=your-n8n-password
N8N_DB_PASSWORD=your-db-password
N8N_ENCRYPTION_KEY=your-encryption-key
```

---

## Git Repository Management

### Proper .gitignore Patterns

**Root `.gitignore` (Project-level):**
- Environment files (`.env*`)
- IDE files (`.vscode/`, `.idea/`)
- OS files (`.DS_Store`, `Thumbs.db`)
- Logs and temporary files
- Compressed files

**Service-Specific Patterns:**

**UI Service:**
- `node_modules/`
- `.next/` build outputs
- TypeScript cache
- ESLint cache

**Supabase Service:**
- ✅ **Tracks:** Schema files (`volumes/db/*.sql`)
- ✅ **Tracks:** Init scripts (`volumes/db/init/`)
- ❌ **Ignores:** Runtime data (`volumes/db/data/`)
- ❌ **Ignores:** User-generated dumps

**N8N Service:**
- N8N data directory
- Credentials files
- Workflow backups (optional)

### Essential Tracked Files

**Database Schema Files (Critical for Setup):**
```
services/supabase/volumes/db/
├── _supabase.sql      # Core Supabase schema
├── auth.sql           # Authentication setup
├── jwt.sql            # JWT configuration
├── logs.sql           # Logging setup
├── pooler.sql         # Connection pooling
├── realtime.sql       # Realtime configuration
├── roles.sql          # Database roles
├── webhooks.sql       # Webhook configuration
└── init/
    └── data.sql       # Initialization data
```

---

## Development Workflow

### Local Development Setup

**1. Start Services in Order:**
```bash
# Terminal 1: Supabase
cd services/supabase
docker-compose up

# Terminal 2: N8N
cd services/n8n
docker-compose up

# Terminal 3: UI Development
cd services/ui
npm install
npm run dev  # Hot reloading
```

**2. Access Services:**
- UI Application: http://localhost:3000
- Supabase Studio: http://localhost:8000
- N8N Workflows: http://localhost:5678

### Development Commands

**Frontend Development:**
```bash
cd services/ui
npm run dev           # Development server with hot reload
npm run build         # Production build
npm run lint          # Code linting
```

**Database Development:**
- Use Supabase Studio at http://localhost:8000
- Design schemas with RLS policies
- Test APIs via PostgREST documentation
- Manage authentication settings

**Workflow Development:**
- Use N8N interface at http://localhost:5678
- Create workflows with HTTP Request nodes
- Connect to Supabase APIs for data operations

### Debugging and Monitoring

**Service Logs:**
```bash
# View real-time logs
docker-compose logs -f [service-name]

# View specific service logs
cd services/supabase && docker-compose logs auth
cd services/supabase && docker-compose logs db
cd services/n8n && docker-compose logs n8n
```

**Service Status:**
```bash
# Check running containers
docker-compose ps

# Check resource usage
docker stats

# Check networks
docker network ls
```

---

## Current Limitations

### Manual Coordination Required

**Service Dependencies:**
- Supabase must start first (provides database and APIs)
- N8N can start independently but needs Supabase for data operations
- UI needs Supabase for authentication and data

**No Automation:**
- No master deployment script
- No shared network configuration
- No automated environment setup
- No service health monitoring
- No integrated logging

**Environment Management:**
- Each service manages its own environment
- No centralized configuration distribution
- Manual environment variable setup required

### Missing Infrastructure Features

**Deployment:**
- No orchestration script
- No service dependency management
- No automated startup sequence

**Monitoring:**
- No centralized logging
- No health checks
- No service discovery
- No load balancing

**Security:**
- No shared network isolation
- No service-to-service authentication
- Basic environment-based security only

---

## Production Deployment

### VPS Deployment Process

**1. Server Setup:**
```bash
# Clone repository
git clone your-repository
cd aiwah-boilerplate

# Configure environment variables
cd services/supabase
cp .env.example .env
# Edit .env with production values
```

**2. Service Deployment:**
```bash
# Start services manually in order
cd services/supabase
docker-compose up -d

cd ../n8n
docker-compose up -d

cd ../ui
docker-compose up -d
```

**3. Reverse Proxy Setup:**
```nginx
# nginx configuration example
server {
    listen 80;
    server_name your-domain.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Security Considerations

**Access Control:**
```bash
# SSH tunnels for admin interfaces
ssh -L 8000:localhost:8000 user@server  # Supabase Studio
ssh -L 5678:localhost:5678 user@server  # N8N Interface
```

**Environment Security:**
- Strong passwords for all services
- Secure JWT secrets
- Firewall configuration
- Regular security updates

**Data Protection:**
- Database volume backups
- SSL certificate configuration
- Network isolation (when implemented)

---

## Implementation Status

### ✅ Complete Infrastructure
- **Service Isolation:** Each service runs independently
- **Official Components:** Latest Supabase and N8N images
- **Git Management:** Proper ignore patterns and tracked schema files
- **Development Workflow:** Hot reloading and service management
- **Production Capable:** Manual deployment process works

### ✅ Application Foundation
- **Next.js Application:** TypeScript, Tailwind, containerized
- **Supabase Integration:** Client libraries and API connectivity
- **N8N Workflows:** Dedicated database and API access capability
- **Database Schema:** Tracked initialization scripts

### ⚠️ Current Limitations
- **Manual Coordination:** No automated service orchestration
- **No Shared Networks:** Services run independently
- **No Master Script:** Individual service management required
- **Basic Monitoring:** No centralized logging or health checks

### 🔄 Needed Improvements
- **Deployment Automation:** Master script for service coordination
- **Network Configuration:** Shared communication layer
- **Environment Management:** Centralized configuration
- **Monitoring:** Health checks and integrated logging
- **Documentation:** Updated setup guides

---

## Technical Architecture Details

### Service Communication

**Current State:**
- Services communicate via HTTP APIs when needed
- No direct database connections between services
- Manual network configuration

**API Endpoints:**
- Supabase: `http://localhost:8000` (Kong gateway)
- N8N: `http://localhost:5678` (direct access)
- UI: `http://localhost:3000` (Next.js server)

### Database Architecture

**Supabase PostgreSQL:**
- User data and application schemas
- Authentication tables (auth.*)
- Real-time subscriptions
- File storage metadata

**N8N PostgreSQL:**
- Workflow definitions
- Execution history
- Credentials (encrypted)
- Completely isolated from user data

### File System Layout

```
aiwah-boilerplate/
├── services/
│   ├── ui/
│   │   ├── src/app/              # Next.js App Router
│   │   ├── public/               # Static assets
│   │   ├── docker-compose.yml    # UI service
│   │   ├── Dockerfile            # Container build
│   │   ├── package.json          # Dependencies
│   │   ├── next.config.js        # Next.js config
│   │   ├── tailwind.config.js    # Tailwind config
│   │   └── .gitignore            # UI ignore patterns
│   ├── supabase/
│   │   ├── docker-compose.yml    # Full Supabase stack
│   │   ├── .env                  # Configuration
│   │   ├── reset.sh              # Reset utility
│   │   ├── .gitignore            # Supabase ignore patterns
│   │   └── volumes/
│   │       ├── db/               # Database config
│   │       │   ├── *.sql         # Schema files (tracked)
│   │       │   └── init/         # Init scripts (tracked)
│   │       ├── api/              # Kong config
│   │       ├── storage/          # File storage
│   │       └── functions/        # Edge functions
│   └── n8n/
│       ├── docker-compose.yml    # N8N + PostgreSQL
│       └── .gitignore            # N8N ignore patterns
├── docs/
│   └── aiwah-os.md              # This documentation
├── .gitignore                   # Project-level patterns
├── README.md                    # User guide
└── LICENSE                      # MIT license
```

---

## Why This Architecture?

### ✅ Current Strengths

**Official Components:** Uses official Supabase self-hosting and N8N images without modifications

**Service Isolation:** Each service is completely independent and can be developed/deployed separately

**Proper Git Management:** Essential schema files tracked, runtime data properly ignored

**Development Ready:** Hot reloading, easy local development, clear service boundaries

**Production Capable:** Same structure works for production with proper configuration

### ⚠️ Current Challenges

**Manual Coordination:** Requires manual service management and startup sequencing

**No Automation:** Missing deployment scripts and automated environment setup

**Basic Monitoring:** No centralized logging or health monitoring

**Network Isolation:** No shared network configuration for service communication

### 🎯 Future Improvements

The architecture provides a solid foundation that can be enhanced with:
- Master deployment script for automated coordination
- Shared network configuration for better service communication
- Centralized environment management
- Integrated monitoring and logging
- Service health checks and dependency management

This represents a practical, working infrastructure that prioritizes official components and service isolation while acknowledging current limitations and providing a clear path for improvements.




