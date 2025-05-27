# AIWAH OS – Technical Documentation

---

## Overview

AIWAH OS is a self-hosted infrastructure boilerplate built with modular Docker services. It provides a complete backend stack with authentication, database, real-time capabilities, and workflow automation using official Supabase self-hosting and isolated service architecture. Each service runs independently and communicates via APIs through a shared Docker network.

---

## Architecture

### Modular Service Architecture

The system consists of 3 main service groups, each with their own docker-compose.yml and isolated networks:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        AIWAH MODULAR ARCHITECTURE                           │
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
│  │ Internal:       │    │ │   Storage    │ │    │ │ Dedicated   │ │        │
│  │ ui-network      │    │ │   Database   │ │    │ │ PostgreSQL  │ │        │
│  └─────────────────┘    │ │   etc...     │ │    │ └─────────────┘ │        │
│           │              │ └──────────────┘ │    │                 │        │
│           │              │                  │    │ Internal:       │        │
│           │              │ Internal:        │    │ n8n-internal    │        │
│           │              │ supabase-internal│    └─────────────────┘        │
│           │              └──────────────────┘              │                │
│           │                       │                       │                │
│           └───────────────────────┼───────────────────────┘                │
│                                   │                                        │
│                    ┌──────────────▼──────────────┐                         │
│                    │     aiwah-network           │                         │
│                    │   (Shared Communication)    │                         │
│                    │   External Network          │                         │
│                    └─────────────────────────────┘                         │
│                                   │                                        │
│              ┌────────────────────▼────────────────────┐                   │
│              │         API Communication               │                   │
│              │  • UI → Supabase (HTTP/WebSocket)      │                   │
│              │  • N8N → Supabase (HTTP APIs)          │                   │
│              │  • No direct database connections      │                   │
│              └─────────────────────────────────────────┘                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Technology Stack

**Infrastructure:**
- Docker Compose (3 isolated service stacks)
- Official Supabase self-hosting (latest stable)
- Shared external network for API communication
- Individual internal networks for service isolation

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
- API-based integration with Supabase

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
└── public/              # Static assets
```

**Configuration:**
- **Container:** `aiwah-ui`
- **Port:** 3000
- **Networks:** `aiwah-network` (external), internal UI network
- **Environment:** Supabase URL and API keys
- **Build:** Custom Dockerfile with Next.js optimization

### 2. Supabase Service (`services/supabase/`)

**Structure:**
```
services/supabase/
├── docker-compose.yml    # Official Supabase stack
└── volumes/             # Configuration and data
    ├── api/             # Kong configuration
    ├── db/              # Database initialization
    ├── storage/         # File storage
    ├── functions/       # Edge functions
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

**Networks:**
- **supabase-internal:** Internal service communication
- **aiwah-network:** External API access (Kong only)

### 3. N8N Service (`services/n8n/`)

**Structure:**
```
services/n8n/
└── docker-compose.yml    # N8N + dedicated PostgreSQL
```

**Services:**
- **n8n:** Workflow engine container
- **n8n-db:** Dedicated PostgreSQL database

**Networks:**
- **n8n-internal:** Internal N8N communication
- **aiwah-network:** External API access

**Isolation:**
- Completely separate from Supabase user data
- Own PostgreSQL instance for workflow data
- Connects to Supabase via HTTP APIs only

---

## Deployment Management

### Master Deploy Script (`deploy.sh`)

The deployment script provides complete lifecycle management:

```bash
# Service Management
./deploy.sh start [service]     # Start all or specific service
./deploy.sh stop [service]      # Stop all or specific service
./deploy.sh restart [service]   # Restart all or specific service

# Monitoring
./deploy.sh status [service]    # Show service status
./deploy.sh logs <service>      # Follow service logs

# Maintenance
./deploy.sh clean              # Remove all containers and networks
```

### Deployment Process

1. **Network Creation:** Creates shared `aiwah-network` if not exists
2. **Environment Setup:** Generates `.env` if missing, copies to services
3. **Service Startup:** Starts services in order (Supabase → N8N → UI)
4. **Health Monitoring:** Provides status and log access

### Environment Configuration

**Master `.env` file (root directory):**
```bash
# Supabase Configuration
POSTGRES_PASSWORD=secure-password
JWT_SECRET=jwt-secret-key
ANON_KEY=supabase-anon-key
SERVICE_ROLE_KEY=supabase-service-key

# Admin Access
DASHBOARD_USERNAME=admin
DASHBOARD_PASSWORD=admin-password

# N8N Configuration
N8N_USERNAME=admin
N8N_PASSWORD=n8n-password
N8N_DB_PASSWORD=n8n-db-password
N8N_ENCRYPTION_KEY=encryption-key

# Service URLs
SUPABASE_URL=http://localhost:8000
SITE_URL=http://localhost:3000
```

The deploy script automatically copies this to each service directory.

---

## Network Architecture

### Network Isolation

**Service-Level Isolation:**
- Each service has its own internal Docker network
- Services cannot directly access each other's databases
- All communication happens via HTTP APIs

**Shared Communication:**
- External `aiwah-network` for API communication
- Only specific services exposed to shared network
- Kong gateway acts as single entry point for Supabase

### API Communication Flow

```
UI Service (port 3000)
    ↓ HTTP/WebSocket
Kong Gateway (port 8000) ← aiwah-network
    ↓ Internal routing
Supabase Services (internal network)
    ↓ Database queries
PostgreSQL (internal only)

N8N Service (port 5678)
    ↓ HTTP APIs
Kong Gateway (port 8000) ← aiwah-network
    ↓ Internal routing
Supabase Services (internal network)
```

---

## Security Implementation

### Network Security
- Service isolation via separate Docker networks
- No direct database access between services
- Kong gateway as single API entry point
- Internal service communication only

### Authentication & Authorization
- Official Supabase JWT implementation
- Kong-managed API routing and security
- Role-based access control (RLS)
- Service-to-service API authentication

### Data Isolation
- Supabase database for user data
- Dedicated N8N database for workflow data
- No shared database access
- API-based data exchange only

---

## File Structure

```
aiwah-boilerplate/
├── deploy.sh                    # Master deployment script
├── .env                         # Master environment configuration
├── services/                    # Isolated service definitions
│   ├── ui/                      # Next.js Frontend Service
│   │   ├── docker-compose.yml   # UI service definition
│   │   ├── Dockerfile           # Container build
│   │   ├── package.json         # Dependencies
│   │   ├── next.config.js       # Next.js configuration
│   │   ├── tailwind.config.js   # Tailwind CSS config
│   │   ├── src/                 # Source code
│   │   │   └── app/             # Next.js App Router
│   │   └── public/              # Static assets
│   ├── n8n/                     # N8N Workflow Service
│   │   └── docker-compose.yml   # N8N + dedicated PostgreSQL
│   └── supabase/                # Official Supabase Stack
│       ├── docker-compose.yml   # All Supabase services
│       └── volumes/             # Configuration & data
│           ├── api/             # Kong configuration
│           ├── db/              # Database initialization
│           ├── storage/         # File storage
│           ├── functions/       # Edge functions
│           ├── logs/            # Log configuration
│           └── pooler/          # Connection pooler config
├── docs/                        # Documentation
│   └── aiwah-os.md             # This technical guide
├── README.md                    # Project overview
└── LICENSE                      # MIT license
```

---

## Development Workflow

### Local Development Setup

1. **Start all services:**
   ```bash
   ./deploy.sh start
   ```

2. **Access services:**
   - UI Application: http://localhost:3000
   - Supabase Studio: http://localhost:8000
   - N8N Workflows: http://localhost:5678

### Development Commands

```bash
# Service management
./deploy.sh start supabase    # Start only Supabase
./deploy.sh logs ui           # View UI logs
./deploy.sh restart n8n       # Restart N8N service

# Development workflow
cd services/ui && npm run dev # UI development with hot reload
./deploy.sh logs supabase     # Debug Supabase issues
./deploy.sh status            # Check all service health
```

### Service Development

**UI Development:**
- Modify `services/ui/src/` for frontend changes
- Use `npm run dev` for hot reloading
- Connect to Supabase via HTTP APIs

**Database Development:**
- Use Supabase Studio at http://localhost:8000
- Design schemas with RLS policies
- Test APIs via PostgREST

**Workflow Development:**
- Use N8N interface at http://localhost:5678
- Create workflows that call Supabase APIs
- Dedicated database for workflow state

---

## Production Deployment

### VPS Deployment

The same modular architecture works for production:

```bash
# On production VPS
git clone your-repository
cd aiwah-boilerplate
./deploy.sh start
```

### Security Considerations

**Network Security:**
- Services isolated by Docker networks
- API-only communication between services
- Kong gateway as security layer

**Access Control:**
- SSH tunnels for admin interface access
- Environment-based credential management
- Official Supabase security features

**Production Setup:**
```bash
# SSH tunnels for admin access
ssh -L 8000:localhost:8000 user@vps  # Supabase Studio
ssh -L 5678:localhost:5678 user@vps  # N8N Interface

# Public UI access via reverse proxy
nginx/caddy → localhost:3000
```

---

## Current Implementation Status

### Complete Infrastructure
✅ **Modular Service Architecture:** 3 isolated service stacks  
✅ **Official Supabase:** Latest stable self-hosting setup  
✅ **Network Isolation:** Separate internal networks + shared communication  
✅ **API Communication:** HTTP-based service interaction  
✅ **Master Deployment:** Single script for all lifecycle management  
✅ **Environment Management:** Centralized configuration distribution  

### Application Foundation
✅ **Next.js Application:** TypeScript, Tailwind, containerized  
✅ **Supabase Integration:** Client libraries and API connectivity  
✅ **N8N Workflows:** Dedicated database and Supabase API access  
✅ **Development Workflow:** Hot reloading and service management  

### Production Ready
✅ **Security:** Network isolation and API-based communication  
✅ **Scalability:** Independent service scaling and deployment  
✅ **Monitoring:** Service status and log management  
✅ **Maintenance:** Clean service lifecycle management  

The codebase provides a complete, production-ready infrastructure with clean service separation, official components, and comprehensive management tooling. Each service can be developed, deployed, and scaled independently while maintaining secure API-based communication.

---

## Why This Architecture?

**✅ Clean Separation:** Each service is completely isolated with its own docker-compose.yml  
**✅ Official Components:** Uses official Supabase self-hosting, not custom implementations  
**✅ API Communication:** Services communicate via HTTP APIs, not direct database access  
**✅ Easy Management:** Single deploy.sh script manages entire infrastructure  
**✅ Production Ready:** Same structure works for development and production  
**✅ Modular Development:** Add/remove/modify services without affecting others  
**✅ Security:** Network isolation with controlled API access points  
**✅ Scalability:** Independent service scaling and resource allocation  

This architecture provides the best of both worlds: the simplicity of a monolithic deployment script with the flexibility and security of microservices architecture.




