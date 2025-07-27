# Choose Your Own Adventure - Documentation

This documentation provides comprehensive information about the Choose Your Own Adventure server application architecture, implementation, and usage.

## Documentation Structure

### 📋 [Architecture Overview](./ARCHITECTURE.md)
- High-level system architecture
- Technology stack and design principles
- Component organization and layered structure
- Key architectural decisions

### 🔄 [Code Flow Documentation](./CODE_FLOW.md)
- Detailed step-by-step application flow
- Story creation and retrieval processes
- Background job processing
- Database transaction patterns
- Error handling and session management

### 🌐 [API Documentation](./API_DOCUMENTATION.md)
- Complete REST API reference
- Request/response schemas
- Usage examples and patterns
- Error handling and status codes
- Interactive documentation links

### 🗄️ [Database Schema](./DATABASE_SCHEMA.md)
- Complete database schema documentation
- Table structures and relationships
- Data flow and constraints
- Performance considerations and indexing
- Transaction management patterns

## Quick Start

1. **Architecture**: Start with [ARCHITECTURE.md](./ARCHITECTURE.md) for system overview
2. **Implementation**: Review [CODE_FLOW.md](./CODE_FLOW.md) for detailed implementation
3. **API Usage**: Reference [API_DOCUMENTATION.md](./API_DOCUMENTATION.md) for endpoint details
4. **Database**: Check [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md) for data structure

## Key Features Documented

- **AI-Powered Story Generation**: LangChain integration with OpenAI
- **Asynchronous Processing**: Background job system for story creation
- **Session Management**: Cookie-based user session tracking
- **Tree-Structured Stories**: Branching narrative with multiple endings
- **RESTful API**: Complete FastAPI implementation with validation

## Development Resources

- **Interactive API Docs**: Available at `/docs` and `/redoc` when server is running
- **Environment Setup**: Documented in main project README
- **Testing**: Patterns and examples throughout documentation
- **Deployment**: Configuration details in architecture documentation

## Contributing

When updating the codebase, please ensure documentation stays current:

1. Update relevant documentation files for architectural changes
2. Add new API endpoints to API documentation
3. Document database schema changes
4. Update code flow for new features
5. Include examples in documentation updates