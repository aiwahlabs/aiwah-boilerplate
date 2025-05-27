# Aiwah Boilerplate

A self-hosted, open-source infrastructure boilerplate for building internal business applications. Provides a complete backend stack with authentication, database, real-time capabilities, and workflow automation - all running as isolated, modular services.

**Current State:** Production-ready infrastructure with clean service separation and API-based communication.

## What You Get

**🏗️ Modular Service Architecture:**
1. **Official Supabase Stack** - Complete self-hosted Supabase with all services
2. **Next.js Application** - TypeScript, Tailwind CSS, containerized frontend
3. **N8N Workflow Engine** - Visual automation with dedicated PostgreSQL database
4. **Master Deployment Script** - One command to rule them all

**🚀 Production-Ready Features:**
- Official Supabase self-hosting (latest stable)
- JWT authentication with proper token management
- Kong API gateway with security and routing
- Isolated services communicating via APIs
- Docker network isolation with shared communication layer
- Environment configuration with secure credential management
- One-script deployment and management

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        AIWAH MODULAR ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐        │
│  │   Next.js UI    │    │   Supabase       │    │   N8N Workflow  │        │
│  │   Port: 3000    │◄──►│   Studio         │    │   Engine        │        │
│  │   (Isolated)    │    │   Port: 8000     │    │   Port: 5678    │        │
│  └─────────────────┘    └──────────────────┘    └─────────────────┘        │
│           │                       │                       │                │
│           └───────────────────────┼───────────────────────┘                │
│                                   │                                        │
│                    ┌──────────────▼──────────────┐                         │
│                    │     aiwah-network           │                         │
│                    │   (Shared Communication)    │                         │
│                    └─────────────────────────────┘                         │
│                                   │                                        │
│              ┌────────────────────▼────────────────────┐                   │
│              │      Official Supabase Services         │                   │
│              │  • Kong Gateway (API Router)            │                   │
│              │  • PostgreSQL Database                  │                   │
│              │  • Auth, REST, Realtime, Storage        │                   │
│              │  • Studio, Meta, Functions, Analytics   │                   │
│              └─────────────────────────────────────────┘                   │
│                                   │                                        │
│              ┌────────────────────▼────────────────────┐                   │
│              │      N8N Dedicated Database             │                   │
│              │      (Completely Isolated)              │                   │
│              └─────────────────────────────────────────┘                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Project Structure

```
aiwah-boilerplate/
├── deploy.sh                    # Master deployment script
├── services/                    # Isolated service definitions
│   ├── ui/                      # Next.js Frontend Service
│   │   ├── docker-compose.yml   # UI service definition
│   │   ├── Dockerfile           # Container build
│   │   ├── src/                 # Source code
│   │   └── package.json         # Dependencies
│   ├── n8n/                     # N8N Workflow Service
│   │   └── docker-compose.yml   # N8N + dedicated PostgreSQL
│   └── supabase/                # Official Supabase Stack
│       ├── docker-compose.yml   # All Supabase services
│       └── volumes/             # Configuration & data
├── docs/                        # Documentation
└── README.md                    # This file
```

## Quick Start

1. **Clone and setup:**
   ```bash
   git clone https://github.com/yourusername/aiwah-boilerplate.git
   cd aiwah-boilerplate
   ```

2. **Start everything:**
   ```bash
   ./deploy.sh start
   ```

3. **Access your services:**
   - **UI Application**: http://localhost:3000
   - **Supabase Studio**: http://localhost:8000 (admin/password)
   - **N8N Workflows**: http://localhost:5678 (admin/password)

## Service Management

The `deploy.sh` script provides complete service lifecycle management:

```bash
# Start all services
./deploy.sh start

# Start specific service
./deploy.sh start supabase
./deploy.sh start ui
./deploy.sh start n8n

# Check status
./deploy.sh status

# View logs
./deploy.sh logs supabase
./deploy.sh logs ui
./deploy.sh logs n8n

# Stop services
./deploy.sh stop

# Restart services
./deploy.sh restart

# Clean everything
./deploy.sh clean
```

## Service Isolation

Each service runs in its own isolated environment:

**UI Service (`services/ui/`):**
- Next.js application with TypeScript and Tailwind
- Builds its own Docker image
- Connects to Supabase via HTTP APIs
- Isolated internal network + shared communication network

**Supabase Service (`services/supabase/`):**
- Official Supabase self-hosting stack
- All services: Kong, Auth, REST, Realtime, Storage, Studio, etc.
- Internal network for Supabase services
- Kong gateway exposed to shared network for API access

**N8N Service (`services/n8n/`):**
- N8N workflow engine + dedicated PostgreSQL
- Completely isolated from user data
- Can connect to Supabase via APIs
- Own internal network + shared communication network

## Development Workflow

**Frontend Development:**
```bash
cd services/ui
npm run dev  # Hot reloading development
```

**Database Design:**
- Use Supabase Studio at http://localhost:8000
- Design schemas, set up RLS policies
- Manage users and authentication

**Workflow Automation:**
- Use N8N at http://localhost:5678
- Create workflows that interact with Supabase APIs
- Dedicated database for workflow data

**Service Development:**
```bash
./deploy.sh logs ui        # Debug UI issues
./deploy.sh restart supabase  # Restart after config changes
./deploy.sh status         # Check all service health
```

## Environment Configuration

The deployment script automatically:
1. Creates a shared Docker network (`aiwah-network`)
2. Generates a master `.env` file if none exists
3. Copies environment variables to each service
4. Starts services in the correct order (Supabase → N8N → UI)

**Manual Configuration:**
Create `.env` in the root directory with:
```bash
# Supabase Configuration
POSTGRES_PASSWORD=your-secure-password
JWT_SECRET=your-jwt-secret
ANON_KEY=your-anon-key
SERVICE_ROLE_KEY=your-service-role-key

# Admin Credentials
DASHBOARD_USERNAME=admin
DASHBOARD_PASSWORD=your-admin-password

# N8N Configuration
N8N_USERNAME=admin
N8N_PASSWORD=your-n8n-password
N8N_DB_PASSWORD=your-n8n-db-password
N8N_ENCRYPTION_KEY=your-encryption-key

# URLs
SUPABASE_URL=http://localhost:8000
SITE_URL=http://localhost:3000
```

## Security Features

**Network Isolation:**
- Each service has its own internal Docker network
- Services communicate via shared `aiwah-network`
- No direct database access between services

**API-Based Communication:**
- UI connects to Supabase via Kong gateway
- N8N connects to Supabase via HTTP APIs
- Clean separation of concerns

**Official Supabase Security:**
- Latest official Supabase images
- Proper JWT token management
- Row Level Security (RLS) policies
- Kong gateway with security plugins

## Production Deployment

The same architecture works for production:

1. **VPS Setup:**
   ```bash
   # On your VPS
   git clone your-repo
   cd aiwah-boilerplate
   ./deploy.sh start
   ```

2. **Access Admin Interfaces:**
   ```bash
   # SSH tunnels for secure access
   ssh -L 8000:localhost:8000 user@your-vps  # Supabase Studio
   ssh -L 5678:localhost:5678 user@your-vps  # N8N
   ```

3. **Public Access:**
   - Set up reverse proxy (nginx/caddy) for port 3000 (UI)
   - Configure SSL certificates
   - Update environment variables for production URLs

## Next Steps

1. **Build Your Application:**
   - Implement authentication in the UI
   - Design your database schema in Supabase Studio
   - Create business logic workflows in N8N

2. **Customize Services:**
   - Modify `services/ui/` for your frontend needs
   - Add custom Supabase functions
   - Create complex N8N workflows

3. **Scale and Deploy:**
   - Use the same structure for production
   - Add monitoring and backups
   - Implement CI/CD pipelines

## Why This Architecture?

**✅ Clean Separation:** Each service is isolated and can be developed/deployed independently  
**✅ Official Components:** Uses official Supabase self-hosting, not custom implementations  
**✅ API Communication:** Services talk via HTTP APIs, not direct database connections  
**✅ Easy Management:** One script to start/stop/monitor everything  
**✅ Production Ready:** Same structure works for development and production  
**✅ Modular:** Add/remove services without affecting others  

## License

MIT License - see [LICENSE](LICENSE) for details.