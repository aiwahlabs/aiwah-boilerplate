# n8n Workflows and Custom Nodes

This directory is intended for n8n workflow templates and custom node implementations for the Aiwah boilerplate.

## Current State

**Empty - Ready for Development**

The n8n workflow engine is running and accessible at http://localhost:5678 (admin/admin123), but no custom workflows or nodes have been created yet.

## What Goes Here

### Workflow Templates (`workflows/`)
- Business process automation examples
- Chat integration workflows
- Database operation templates
- External service integrations
- Scheduled automation tasks

### Custom Nodes (`nodes/`)
- Supabase-specific operations
- Chat system integrations
- Business logic components
- AI tool integrations

### Configuration (`config/`)
- Environment-specific settings
- Custom node configurations
- Workflow deployment scripts

## Getting Started

1. **Access n8n Interface**: http://localhost:5678
   - Username: `admin`
   - Password: `admin123`

2. **Create Your First Workflow**:
   - Connect to the PostgreSQL database (connection details in docker-compose.yml)
   - Build workflows that interact with your Supabase tables
   - Export workflows to this directory for version control

3. **Database Connection Settings**:
   - Host: `db`
   - Port: `5432`
   - Database: `postgres`
   - Username: `postgres`
   - Password: Use `POSTGRES_PASSWORD` from your .env file

## Example Workflow Ideas

- **User Onboarding**: Automatically create user profiles and send welcome messages
- **Chat Moderation**: Monitor messages for inappropriate content
- **Data Sync**: Sync data between different systems
- **Notifications**: Send alerts based on database changes
- **Reports**: Generate and email periodic reports
- **Backup**: Automated database and file backups

## Development Workflow

1. Create workflows in the n8n web interface
2. Export workflows as JSON files to `workflows/` directory
3. Version control your workflows alongside your code
4. Import workflows on new deployments

## Resources

- [n8n Documentation](https://docs.n8n.io/)
- [n8n Community Workflows](https://n8n.io/workflows/)
- [PostgreSQL Node Documentation](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.postgres/) 