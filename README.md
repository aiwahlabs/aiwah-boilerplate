# Aiwah Boilerplate

A self-hosted infrastructure boilerplate for building applications with authentication, database, and workflow automation. Built with Docker Compose using Supabase Cloud, N8N workflow engine, and Appsmith no-code platform.

**Current State:** Complete business operating system with three services plus cloud database.

## What You Get

**🏗️ Three Services + Cloud Database:**
1. **Supabase Cloud** - Managed Supabase with all services (auth, database, storage, etc.)
2. **Next.js Application** - TypeScript frontend with Supabase Cloud integration
3. **N8N Workflow Engine** - Visual automation with dedicated PostgreSQL database
4. **Appsmith** - Open-source no-code platform for internal tools and admin panels

**🔧 Current Setup:**
- Three independent Docker services
- Supabase Cloud for authentication and database
- Complete no-code business automation stack
- Basic environment configuration
- Service-specific `.gitignore` files with proper patterns

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           CURRENT ARCHITECTURE                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐  ┌─────────────────────┐   │
│  │  Next.js    │  │  Supabase    │  │    N8N      │  │     Appsmith        │   │
│  │  Port: 3000 │◄►│  Cloud       │◄►│ Port: 5678  │◄►│   Port: 8080        │   │
│  │             │  │ (External)   │  │             │  │                     │   │
│  │ Custom UI   │  │ Database &   │  │ Workflow    │  │ No-Code Internal    │   │
│  │ Frontend    │  │ Auth Service │  │ Automation  │  │ Tools & Dashboards  │   │
│  └─────────────┘  └──────────────┘  └─────────────┘  └─────────────────────┘   │
│                                                                                 │
│  Complete Business Operating System: UI + Database + Automation + Tools        │
└─────────────────────────────────────────────────────────────────────────────────┘
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
│   │   ├── env.example          # Environment template
│   │   └── .gitignore           # UI-specific ignore patterns
│   ├── n8n/                     # N8N Workflow Service
│   │   ├── docker-compose.yml   # N8N + dedicated PostgreSQL
│   │   ├── env.example          # Environment template
│   │   └── .gitignore           # N8N-specific ignore patterns
│   └── appsmith/                # Appsmith No-Code Platform
│       ├── docker-compose.yml   # Appsmith Community Edition
│       ├── env.example          # Environment template
│       └── .gitignore           # Appsmith-specific ignore patterns
├── docs/                        # Documentation
├── .gitignore                   # Project-level ignore patterns
└── README.md                    # This file
```

## Quick Start

### Prerequisites
1. **Supabase Cloud Account**: Sign up at [supabase.com](https://supabase.com)
2. **Create a new Supabase project** and note your:
   - Project URL
   - Anon (public) key
   - Service role key (for server-side operations)

### Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/yourusername/aiwah-boilerplate.git
   cd aiwah-boilerplate
   ```

2. **Configure environment variables:**
   ```bash
   # Create environment file for UI service
   cd services/ui
   cp env.example .env.local  # Fill in your Supabase credentials
   
   # Create environment file for N8N service
   cd ../n8n
   cp env.example .env  # Fill in N8N credentials
   
   # Create environment file for Appsmith service
   cd ../appsmith
   cp env.example .env  # Fill in Appsmith credentials (optional)
   ```

3. **Start all services:**
   ```bash
   # Start N8N
   cd services/n8n
   docker-compose up -d
   
   # Start Appsmith
   cd ../appsmith
   docker-compose up -d
   
   # Start UI
   cd ../ui
   docker-compose up -d
   ```

4. **Access your services:**
   - **UI Application**: http://localhost:3000
   - **Supabase Dashboard**: https://app.supabase.com (your cloud project)
   - **N8N Workflows**: http://localhost:5678
   - **Appsmith Platform**: http://localhost:8080

## Service Management

Each service can be managed independently:

**N8N Service:**
```bash
cd services/n8n
docker-compose up -d        # Start
docker-compose down         # Stop
docker-compose logs -f      # View logs
```

**Appsmith Service:**
```bash
cd services/appsmith
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
cd ../appsmith && docker-compose down -v

# Clean up Docker
docker system prune -f
docker volume prune -f
```

## Service Details

### Supabase Cloud

**What's Included:**
- PostgreSQL database with all extensions
- GoTrue authentication service
- PostgREST API generator
- Realtime WebSocket service
- Storage service for files
- Supabase Studio (admin interface)
- Edge functions runtime
- Analytics and logging
- **Managed hosting and backups**

**Configuration:**
- Access via Supabase Cloud dashboard
- Environment variables for connection
- No local infrastructure required

### UI Service (`services/ui/`)

**What's Included:**
- Next.js 13+ with App Router
- TypeScript configuration
- Tailwind CSS for styling
- Supabase client integration (Cloud)
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

### Appsmith Service (`services/appsmith/`)

**What's Included:**
- Appsmith Community Edition (open-source)
- Built-in MongoDB for app data
- Drag-and-drop app builder
- 80+ pre-built UI components
- Database connectors and API integrations
- Git version control support

**Access:**
- URL: http://localhost:8080
- First-time setup: Create admin account on first visit

## Environment Configuration

**UI Service Environment (`services/ui/env.example`):**
```bash
# Supabase Cloud Configuration
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key

# Optional: Service role key for server-side operations
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
```

**N8N Configuration (`services/n8n/env.example`):**
```bash
# N8N Authentication
N8N_USERNAME=admin
N8N_PASSWORD=your-n8n-password

# N8N Database
N8N_DB_PASSWORD=your-db-password

# N8N Security
N8N_ENCRYPTION_KEY=your-encryption-key
```

**Appsmith Configuration (`services/appsmith/env.example`):**
```bash
# Appsmith Security (optional - auto-generated if not provided)
APPSMITH_ENCRYPTION_PASSWORD=your-encryption-password
APPSMITH_ENCRYPTION_SALT=your-encryption-salt

# Optional: Email and OAuth configuration
# See env.example for full options
```

## Development Workflow

**Database Design:**
1. Access Supabase Cloud dashboard at https://app.supabase.com
2. Design schemas and set up Row Level Security (RLS) policies
3. Manage users and authentication settings
4. Test APIs via the built-in API documentation

**Frontend Development:**
1. Modify files in `services/ui/src/`
2. Use `npm run dev` for hot reloading
3. Connect to Supabase Cloud using the official client libraries

**Workflow Automation:**
1. Access N8N at http://localhost:5678
2. Create workflows that can interact with Supabase Cloud APIs
3. Use HTTP Request nodes to call Supabase endpoints
4. Trigger workflows from Appsmith apps or UI events

**No-Code App Development:**
1. Access Appsmith at http://localhost:8080
2. Create admin panels, dashboards, and internal tools
3. Connect to Supabase Cloud databases and APIs
4. Build forms, tables, and business logic without coding
5. Integrate with N8N workflows for automation

## Integration Examples

**Appsmith ↔ Supabase:**
- Connect Appsmith to your Supabase Cloud database
- Build admin panels for data management
- Create user dashboards and reporting tools

**N8N ↔ Supabase:**
- Trigger workflows on database changes
- Automate data processing and notifications
- Sync data between external services

**Appsmith ↔ N8N:**
- Trigger N8N workflows from Appsmith buttons
- Display workflow results in Appsmith dashboards
- Create approval flows and business processes

## Important Files Tracked in Git

The project properly tracks essential configuration files:

**Configuration Files (tracked):**
- `services/ui/package.json` - UI dependencies
- `services/n8n/docker-compose.yml` - N8N service configuration
- `services/appsmith/docker-compose.yml` - Appsmith service configuration
- `services/*/env.example` - Environment templates

**What's Ignored:**
- `.env*` files - Sensitive configuration
- `node_modules/` - Dependencies
- `volumes/` - Runtime data
- `stacks/` - Appsmith runtime data
- Log files and temporary data

## Current Benefits

**Complete Business Operating System:**
- **Frontend**: Custom Next.js applications
- **Backend**: Supabase Cloud database and authentication
- **Automation**: N8N workflow engine
- **Internal Tools**: Appsmith no-code platform

**Cloud-Native Architecture:**
- No local database management required
- Supabase Cloud handles scaling, backups, and maintenance
- Reduced local resource usage
- Faster setup and deployment

**Developer & Business User Friendly:**
- Developers: Full code control with Next.js and N8N
- Business Users: No-code tools with Appsmith
- Automatic backups and point-in-time recovery
- Professional monitoring and alerting

## Production Deployment

**VPS Setup:**
1. Clone the repository on your server
2. Configure environment variables for each service
3. Set up your Supabase Cloud project
4. Start services with docker-compose
5. Set up reverse proxy for public access (nginx/caddy)
6. Configure SSL certificates

**Security Considerations:**
- Use environment variables for all secrets
- Configure proper RLS policies in Supabase
- Use strong passwords for N8N and Appsmith
- Configure proper firewall rules
- Regular security updates for Docker images

## Supabase Cloud Setup

**Getting Started:**
1. Sign up at [supabase.com](https://supabase.com)
2. Create a new project
3. Copy your project URL and API keys
4. Configure your environment variables
5. Design your database schema in the Supabase dashboard

**Key Features:**
- **Authentication**: Built-in user management with multiple providers
- **Database**: PostgreSQL with real-time subscriptions
- **Storage**: File uploads and management
- **Edge Functions**: Serverless functions at the edge
- **APIs**: Auto-generated REST and GraphQL APIs

## Next Steps

**Application Development:**
1. Implement authentication in the UI using Supabase Auth
2. Design your database schema in Supabase Cloud dashboard
3. Create business logic workflows in N8N
4. Build internal tools and dashboards in Appsmith
5. Set up Supabase Edge Functions for custom backend logic

**Advanced Features:**
1. Implement real-time features using Supabase subscriptions
2. Add file upload functionality using Supabase Storage
3. Create automated workflows that trigger on database changes
4. Build approval processes connecting Appsmith and N8N
5. Implement advanced authentication flows

**Integration Workflows:**
1. **Data Entry**: Appsmith forms → Supabase database → N8N processing
2. **Notifications**: Database changes → N8N workflows → Email/Slack alerts
3. **Reporting**: Supabase data → Appsmith dashboards → Automated reports
4. **User Management**: Supabase Auth → Appsmith admin panels → N8N onboarding

## Why This Setup?

**✅ Complete Business Solution:** Frontend + Database + Automation + Internal Tools  
**✅ Cloud-Native:** Uses managed Supabase Cloud for reliability and scale  
**✅ No-Code Friendly:** Appsmith enables business users to build tools  
**✅ Developer Friendly:** Full code control when needed  
**✅ Production Ready:** Same structure works for production deployment  
**✅ Cost Effective:** Pay only for what you use with Supabase Cloud  
**✅ Scalable:** All components handle growth automatically  

**🎯 Current State:** Complete business operating system ready for any application type - from simple internal tools to complex multi-user applications.

## License

MIT License - see [LICENSE](LICENSE) for details.