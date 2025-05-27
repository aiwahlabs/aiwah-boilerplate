# Aiwah Boilerplate

A self-hosted infrastructure boilerplate for building applications with authentication, database, and workflow automation. Built with Docker Compose using official Supabase self-hosting and N8N workflow engine.

**Current State:** Basic infrastructure setup with three independent services that need manual coordination.

## What You Get

**🏗️ Three Independent Services:**
1. **Supabase Stack** - Official self-hosted Supabase with all services (auth, database, storage, etc.)
2. **Next.js Application** - TypeScript frontend with Supabase integration
3. **N8N Workflow Engine** - Visual automation with dedicated PostgreSQL database

**🔧 Current Setup:**
- Each service has its own `docker-compose.yml`
- Manual service management required
- Basic environment configuration
- Service-specific `.gitignore` files with proper patterns

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    CURRENT ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐    ┌──────────────────┐    ┌─────────────┐ │
│  │   Next.js UI    │    │   Supabase       │    │   N8N       │ │
│  │   Port: 3000    │    │   Studio         │    │   Port: 5678│ │
│  │                 │    │   Port: 8000     │    │             │ │
│  │ Independent     │    │ Independent      │    │ Independent │ │
│  │ Service         │    │ Service Stack    │    │ Service     │ │
│  └─────────────────┘    └──────────────────┘    └─────────────┘ │
│                                                                 │
│  Each service runs independently with its own docker-compose    │
│  Manual coordination required between services                  │
└─────────────────────────────────────────────────────────────────┘
```

## Project Structure

```
aiwah-boilerplate/
├── services/                    # Independent service definitions
│   ├── ui/                      # Next.js Frontend Service
│   │   ├── docker-compose.yml   # UI service definition
│   │   ├── Dockerfile           # Container build
│   │   ├── src/                 # Source code
│   │   ├── package.json         # Dependencies
│   │   └── .gitignore           # UI-specific ignore patterns
│   ├── n8n/                     # N8N Workflow Service
│   │   ├── docker-compose.yml   # N8N + dedicated PostgreSQL
│   │   └── .gitignore           # N8N-specific ignore patterns
│   └── supabase/                # Official Supabase Stack
│       ├── docker-compose.yml   # All Supabase services
│       ├── volumes/             # Configuration & data
│       │   └── db/              # Database initialization scripts
│       ├── .env                 # Supabase configuration
│       ├── .gitignore           # Supabase-specific ignore patterns
│       └── reset.sh             # Supabase reset utility
├── docs/                        # Documentation
├── .gitignore                   # Project-level ignore patterns
└── README.md                    # This file
```

## Quick Start

1. **Clone the repository:**
   ```bash
   git clone https://github.com/yourusername/aiwah-boilerplate.git
   cd aiwah-boilerplate
   ```

2. **Start Supabase (required first):**
   ```bash
   cd services/supabase
   docker-compose up -d
   ```

3. **Start N8N:**
   ```bash
   cd ../n8n
   docker-compose up -d
   ```

4. **Start UI:**
   ```bash
   cd ../ui
   docker-compose up -d
   ```

5. **Access your services:**
   - **UI Application**: http://localhost:3000
   - **Supabase Studio**: http://localhost:8000
   - **N8N Workflows**: http://localhost:5678

## Service Management

Each service must be managed independently:

**Supabase Service:**
```bash
cd services/supabase
docker-compose up -d        # Start
docker-compose down         # Stop
docker-compose logs -f      # View logs
docker-compose ps           # Check status
```

**N8N Service:**
```bash
cd services/n8n
docker-compose up -d        # Start
docker-compose down         # Stop
docker-compose logs -f      # View logs
```

**UI Service:**
```bash
cd services/ui
docker-compose up -d        # Start
docker-compose down         # Stop
docker-compose logs -f      # View logs
```

**Reset Everything:**
```bash
# Stop all services
cd services/ui && docker-compose down -v
cd ../n8n && docker-compose down -v
cd ../supabase && docker-compose down -v

# Clean up Docker
docker system prune -f
docker volume prune -f
```

## Service Details

### Supabase Service (`services/supabase/`)

**What's Included:**
- PostgreSQL database with all extensions
- GoTrue authentication service
- PostgREST API generator
- Realtime WebSocket service
- Storage service for files
- Supabase Studio (admin interface)
- Kong API gateway
- Edge functions runtime
- Analytics and logging

**Configuration:**
- Environment variables in `services/supabase/.env`
- Database initialization scripts in `volumes/db/`
- Proper `.gitignore` that tracks schema files but ignores runtime data

### UI Service (`services/ui/`)

**What's Included:**
- Next.js 13+ with App Router
- TypeScript configuration
- Tailwind CSS for styling
- Supabase client integration
- Docker containerization

**Development:**
```bash
cd services/ui
npm install
npm run dev  # Hot reloading development server
```

### N8N Service (`services/n8n/`)

**What's Included:**
- N8N workflow engine
- Dedicated PostgreSQL database (separate from Supabase)
- Basic authentication
- Persistent workflow storage

**Access:**
- URL: http://localhost:5678
- Default credentials: Set via environment variables

## Environment Configuration

**Supabase Configuration (`services/supabase/.env`):**
```bash
# Database
POSTGRES_PASSWORD=your-secure-password

# JWT
JWT_SECRET=your-jwt-secret-key
ANON_KEY=your-anon-key
SERVICE_ROLE_KEY=your-service-role-key

# Admin
DASHBOARD_USERNAME=admin
DASHBOARD_PASSWORD=your-admin-password

# URLs
SITE_URL=http://localhost:3000
```

**N8N Configuration:**
Set these in your environment or docker-compose:
```bash
N8N_USERNAME=admin
N8N_PASSWORD=your-n8n-password
N8N_DB_PASSWORD=your-db-password
N8N_ENCRYPTION_KEY=your-encryption-key
```

## Development Workflow

**Database Design:**
1. Access Supabase Studio at http://localhost:8000
2. Design schemas and set up Row Level Security (RLS) policies
3. Manage users and authentication settings
4. Test APIs via the built-in API documentation

**Frontend Development:**
1. Modify files in `services/ui/src/`
2. Use `npm run dev` for hot reloading
3. Connect to Supabase using the official client libraries

**Workflow Automation:**
1. Access N8N at http://localhost:5678
2. Create workflows that can interact with Supabase APIs
3. Use HTTP Request nodes to call Supabase endpoints

## Important Files Tracked in Git

The project now properly tracks essential database initialization files:

**Supabase Schema Files (tracked):**
- `services/supabase/volumes/db/*.sql` - Database schema and configuration
- `services/supabase/volumes/db/init/data.sql` - Initialization data

**What's Ignored:**
- `volumes/db/data/` - Runtime database data
- `node_modules/` - Dependencies
- `.env` files - Sensitive configuration
- Log files and temporary data

## Current Limitations

**Manual Coordination Required:**
- No automated service orchestration
- Services must be started in correct order (Supabase first)
- No shared network configuration
- Environment variables must be managed per service

**Missing Features:**
- No master deployment script
- No automated environment setup
- No service health monitoring
- No integrated logging

## Production Deployment

**VPS Setup:**
1. Clone the repository on your server
2. Configure environment variables for each service
3. Start services manually in order
4. Set up reverse proxy for public access (nginx/caddy)
5. Configure SSL certificates
6. Set up backups for database volumes

**Security Considerations:**
- Change all default passwords
- Use strong JWT secrets
- Configure proper firewall rules
- Use SSH tunnels for admin interface access
- Regular security updates

## Next Steps

**Immediate Improvements Needed:**
1. Create master deployment script
2. Implement shared network configuration
3. Add automated environment setup
4. Implement service health monitoring
5. Add integrated logging solution

**Application Development:**
1. Implement authentication in the UI
2. Design your database schema in Supabase Studio
3. Create business logic workflows in N8N
4. Build your application features

## Why This Setup?

**✅ Official Components:** Uses official Supabase self-hosting and N8N images  
**✅ Service Isolation:** Each service is completely independent  
**✅ Proper Git Patterns:** Essential files tracked, runtime data ignored  
**✅ Development Ready:** Hot reloading and easy local development  
**✅ Production Capable:** Same structure works for production deployment  

**⚠️ Current State:** Basic infrastructure that requires manual coordination but provides a solid foundation for building applications.

## License

MIT License - see [LICENSE](LICENSE) for details.