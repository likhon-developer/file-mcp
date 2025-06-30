# PostgreSQL MCP Server

A Model Context Protocol (MCP) server that provides read-only access to PostgreSQL databases for AI assistants and other MCP clients.

## Features

- **Read-only database access** - Safe querying with built-in protections
- **Schema introspection** - Explore database structure and relationships
- **Data analysis tools** - Built-in analysis functions for common tasks
- **Search capabilities** - Find tables and columns by pattern
- **Comprehensive error handling** - Informative error messages and graceful failures
- **Serverless compatible** - No native dependencies, works on Vercel/Netlify
- **Security first** - SQL injection prevention and query validation

## Installation

### From npm (recommended)

\`\`\`bash
npm install -g postgresql-mcp-server
\`\`\`

### From source

\`\`\`bash
git clone <repository-url>
cd postgresql-mcp-server
npm install
npm run build
npm link
\`\`\`

## Configuration

The server requires a PostgreSQL connection string via the `DATABASE_URL` environment variable:

\`\`\`bash
export DATABASE_URL="postgresql://username:password@hostname:port/database"
\`\`\`

### Supported connection string formats:

- `postgresql://user:pass@host:port/dbname`
- `postgresql://user:pass@host:port/dbname?sslmode=require`
- `postgres://user:pass@host:port/dbname`

### Supported database providers:

- **Neon** - Serverless PostgreSQL
- **Supabase** - Open source Firebase alternative
- **Railway** - Infrastructure platform
- **Render** - Cloud application platform
- **Traditional PostgreSQL** - Self-hosted or cloud instances

## Usage

### With Claude Desktop

Add to your Claude Desktop configuration file:

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

\`\`\`json
{
  "mcpServers": {
    "postgresql": {
      "command": "postgresql-mcp-server",
      "env": {
        "DATABASE_URL": "postgresql://username:password@hostname:port/database"
      }
    }
  }
}
\`\`\`

### With other MCP clients

The server implements the standard MCP protocol and can be used with any compatible client.

## API Reference

### Resources

#### `postgresql://database-schema`
Complete database schema with all tables and columns in JSON format.

#### `postgresql://table-list`
Simple list of all tables in the database.

### Tools

#### `execute_sql`
Execute read-only SQL queries against the database.

**Parameters:**
- `query` (string, required): The SQL query to execute
- `limit` (number, optional): Maximum rows to return (default: 100, max: 1000)

**Example:**
\`\`\`sql
SELECT * FROM users WHERE created_at > '2024-01-01'
\`\`\`

#### `get_table_info`
Get detailed information about a specific table.

**Parameters:**
- `tableName` (string, required): Name of the table to analyze

**Returns:**
- Table schema and column information
- Row count
- Sample data (first 5 rows)

#### `analyze_data`
Generate common data analysis queries for a table.

**Parameters:**
- `tableName` (string, required): Name of the table to analyze
- `analysisType` (enum, required): Type of analysis
  - `summary`: Basic row counts and statistics
  - `distribution`: Value distribution for first column
  - `nulls`: Null value analysis across columns
  - `duplicates`: Duplicate row detection
  - `trends`: Time-based trend analysis (requires date/timestamp columns)

#### `search_tables`
Search for tables and columns matching a pattern.

**Parameters:**
- `pattern` (string, required): Search pattern (supports wildcards)
- `searchType` (enum, optional): What to search for
  - `tables`: Search table names only
  - `columns`: Search column names only
  - `both`: Search both tables and columns (default)

## Security

This server implements several security measures:

- **Read-only queries only**: Only SELECT, WITH, EXPLAIN, SHOW, and DESCRIBE statements are allowed
- **Query validation**: Dangerous keywords (INSERT, UPDATE, DELETE, DROP, etc.) are blocked
- **Input sanitization**: Table names and parameters are validated to prevent SQL injection
- **Connection limits**: Built-in connection pooling prevents resource exhaustion
- **Error handling**: Detailed errors for debugging without exposing sensitive information

## Development

### Setup

\`\`\`bash
git clone <repository-url>
cd postgresql-mcp-server
npm install
\`\`\`

### Running in development

\`\`\`bash
npm run dev
\`\`\`

### Building

\`\`\`bash
npm run build
\`\`\`

### Testing

\`\`\`bash
# Run tests
npm test

# Run tests with coverage
npm run test:coverage

# Run linting
npm run lint
\`\`\`

### Project Structure

\`\`\`
src/
├── index.ts              # Main server entry point
├── database.ts           # Database connection and query management
├── validation.ts         # Environment validation
├── logging.ts            # Logging utility
├── tools/
│   ├── index.ts          # Tools handler
│   ├── execute-sql.ts    # SQL execution tool
│   ├── get-table-info.ts # Table information tool
│   ├── analyze-data.ts   # Data analysis tool
│   └── search-tables.ts  # Search tool
└── resources/
    ├── index.ts          # Resources handler
    ├── database-schema.ts # Schema resource
    └── table-list.ts     # Table list resource

__tests__/                # Test files
├── database.test.ts
├── tools/
└── resources/
\`\`\`

## Examples

### Basic Queries
\`\`\`sql
-- Get recent users
SELECT * FROM users WHERE created_at > '2024-01-01' LIMIT 10;

-- Count records by status
SELECT status, COUNT(*) FROM orders GROUP BY status;

-- Find top products
SELECT product_name, SUM(quantity) as total_sold 
FROM order_items 
GROUP BY product_name 
ORDER BY total_sold DESC 
LIMIT 5;
\`\`\`

### Data Analysis
- **Summary**: Get basic statistics about any table
- **Distribution**: Analyze value distributions in columns
- **Null Analysis**: Find missing data patterns
- **Duplicates**: Identify duplicate records
- **Trends**: Analyze time-based patterns

### Search & Discovery
- Search for tables: `user`, `order`, `product`
- Search for columns: `email`, `created_at`, `status`
- Explore database structure and relationships

## Troubleshooting

### Connection Issues

1. **"Failed to connect to database"**
   - Verify your `DATABASE_URL` is correct
   - Check that the database server is running and accessible
   - Ensure your credentials are valid

2. **SSL/TLS errors**
   - For cloud databases, try adding `?sslmode=require` to your connection string
   - For local development, you may need `?sslmode=disable`

### Query Issues

1. **"Only read-only queries are allowed"**
   - The server only accepts SELECT, WITH, EXPLAIN, SHOW, and DESCRIBE statements
   - Modify your query to use only these statement types

2. **"Query contains potentially dangerous operations"**
   - Your query contains keywords that could modify data
   - Review your query for INSERT, UPDATE, DELETE, DROP, CREATE, ALTER, or TRUNCATE statements

### Performance Issues

1. **Slow queries**
   - Use the `limit` parameter to reduce result set size
   - Add appropriate indexes to your database
   - Consider using the analysis tools instead of raw queries for large datasets

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Run the test suite
6. Submit a pull request

## License

MIT License - see LICENSE file for details.
\`\`\`

```plaintext file=".env.example"
# PostgreSQL Database Connection
# Replace with your actual database connection string
DATABASE_URL=postgresql://username:password@hostname:port/database

# Optional: Enable debug logging
DEBUG=true

# Example connection strings:
# Local PostgreSQL:
# DATABASE_URL=postgresql://postgres:password@localhost:5432/mydb

# Cloud PostgreSQL (with SSL):
# DATABASE_URL=postgresql://user:pass@host.com:5432/dbname?sslmode=require

# Neon (serverless PostgreSQL):
# DATABASE_URL=postgresql://user:pass@ep-xxx.us-east-1.aws.neon.tech/dbname?sslmode=require

# Supabase:
# DATABASE_URL=postgresql://postgres:[password]@db.[project-ref].supabase.co:5432/postgres

# Railway:
# DATABASE_URL=postgresql://postgres:pass@containers-us-west-xxx.railway.app:port/railway

# Render:
# DATABASE_URL=postgresql://user:pass@dpg-xxx-a.oregon-postgres.render.com/dbname
