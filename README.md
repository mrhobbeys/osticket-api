# OSTicket API

<p align="center">
 <img width="350" src="https://bmsvieira.github.io/osticket-api/images/logo.png">
 <br><br>
 <strong>Unofficial OSTicket REST API</strong><br><br>
 A comprehensive PHP-based web service to interact with OSTicket through RESTful API calls.<br>
 The purpose of this API is to help the community and leverage the use of OSTicket.
</p>

<p align="center">
 <a href="#installation">Installation</a> •
 <a href="#configuration">Configuration</a> •
 <a href="#authentication">Authentication</a> •
 <a href="#api-endpoints">API Endpoints</a> •
 <a href="#examples">Examples</a> •
 <a href="#contributing">Contributing</a>
</p>

## Features

- 🔐 **Secure API Key Authentication** with optional IP restriction
- 🎯 **Complete CRUD Operations** for all major OSTicket entities
- 📊 **Advanced Filtering & Sorting** by date ranges and custom parameters  
- 🔍 **Comprehensive Entity Support**: Tickets, Users, Departments, SLA, FAQ, Topics, Tasks
- 📝 **Built-in System Logging** for all API requests
- ⚡ **JSON Response Format** with execution time tracking
- 🛡️ **Input Validation & SQL Injection Protection**
- 🚀 **RESTful Design** following HTTP standards

## Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/BMSVierira/osticket-api.git
   cd osticket-api
   ```

2. **Upload to your web server**
   - Place the `ost_wbs` folder in your web server directory
   - Ensure PHP 7.0+ is installed with MySQLi extension

3. **Set proper permissions**
   ```bash
   chmod 755 ost_wbs/
   chmod 644 ost_wbs/*.php
   chmod 644 ost_wbs/classes/*.php
   ```

## Configuration

1. **Database Configuration**
   
   Edit `ost_wbs/config.php` with your OSTicket database credentials:
   ```php
   // Database Credentials
   define('DBTYPE','mysql');        // Database type
   define('DBHOST','localhost');    // Database host
   define('DBNAME','osticket_db');  // OSTicket database name
   define('DBUSER','db_username');  // Database username
   define('DBPASS','db_password');  // Database password
   
   // Table prefix (match your OSTicket installation)
   define('TABLE_PREFIX','ost_');
   ```

2. **Security Settings**
   ```php
   // IP Address Restriction (optional)
   define('APIKEY_RESTRICT', false); // Set to true for IP-based access control
   
   // System Logging
   define('WRITE_SYSTEMLOG', true); // Log successful requests to OSTicket system log
   ```

3. **Create API Key in OSTicket**
   - Login to OSTicket Admin Panel
   - Go to `Manage > API Keys`
   - Click "Add New API Key"
   - Configure permissions (Can Create Tickets, Can Execute Cron, etc.)
   - Note the generated 32-character API key

## Authentication

All API requests require authentication via API key in the request header:

```
Content-Type: application/json
ApiKey: your-32--character--api-key-here
```

### Permission Levels
- **Read Only**: Can only retrieve data (GET requests)
- **Create Tickets**: Can create new tickets and perform write operations
- **Execute Cron**: Can perform administrative operations

## API Endpoints

### Request Format
All requests use JSON format with the following structure:
```json
{
  "query": "EntityName",
  "condition": "methodName",
  "sort": "sortParameter",
  "parameters": {
    "param1": "value1",
    "param2": "value2"
  }
}
```

### Response Format
```json
{
  "status": "Success|Error",
  "time": 0.0234,
  "data": {
    "total": 10,
    "entity_name": [...]
  }
}
```

### Available Entities & Methods

#### 🎫 Tickets
| Method | HTTP | Description | Required Parameters |
|--------|------|-------------|-------------------|
| `all` | GET | List tickets by date range and status | `start_date`, `end_date`, `status` |
| `specific` | GET | Get specific ticket by ID or number | `id` |
| `add` | POST/PUT | Create new ticket | `title`, `subject`, `user_id`, `priority_id`, `status_id`, `dept_id`, `sla_id`, `topic_id` |
| `reply` | POST/PUT | Add reply to ticket | `ticket_id`, `message` |
| `close` | POST/PUT | Close ticket | `ticket_id` |

#### 👥 Users  
| Method | HTTP | Description | Required Parameters |
|--------|------|-------------|-------------------|
| `all` | GET | List users by date range | `start_date`, `end_date` |
| `specific` | GET | Get specific user | `id` or `email` |
| `add` | POST/PUT | Create new user | `name`, `email`, `phone`, `org_id`, `default_email_id`, `status`, `timezone`, `password` |

#### 🏢 Departments
| Method | HTTP | Description | Required Parameters |
|--------|------|-------------|-------------------|
| `all` | GET | List departments | `start_date`, `end_date` (for creationDate sort) or none (for name sort) |
| `specific` | GET | Get specific department | `id` |
| `add` | POST/PUT | Create new department | `name`, `ispublic`, `autoresp_email_id`, `manager_id` |

#### ⚡ SLA (Service Level Agreements)
| Method | HTTP | Description | Required Parameters |
|--------|------|-------------|-------------------|
| `all` | GET | List SLAs by date range | `start_date`, `end_date` |
| `specific` | GET | Get specific SLA | `id` |
| `add` | POST/PUT | Create new SLA | `name`, `flags`, `grace_period`, `schedule_id`, `notes` |
| `delete` | DELETE | Delete SLA | `id` |

#### ❓ FAQ
| Method | HTTP | Description | Required Parameters |
|--------|------|-------------|-------------------|
| `all` | GET | List FAQ categories | None required |
| `specific` | GET | Get specific FAQ | `id` |

#### 📋 Topics  
| Method | HTTP | Description | Required Parameters |
|--------|------|-------------|-------------------|
| `all` | GET | List help topics | None required |
| `specific` | GET | Get specific topic | `id` |

#### ✅ Tasks
| Method | HTTP | Description | Required Parameters |
|--------|------|-------------|-------------------|
| `all` | GET | List tasks by date range | `start_date`, `end_date`, `ticket_id` |
| `specific` | GET | Get specific task | `id` |

### Sort Parameters
- `creationDate`: Sort by creation date (requires `start_date` and `end_date`)
- `id`: Sort by ID
- `name`: Sort by name (departments only)

## Examples

### 1. Get Tickets by Date Range and Status
```bash
curl -X GET https://your-domain.com/ost_wbs/ \
  -H "Content-Type: application/json" \
  -H "ApiKey: your-api-key-here" \
  -d '{
    "query": "ticket",
    "condition": "all",
    "sort": "creationDate",
    "parameters": {
      "start_date": "2024-01-01",
      "end_date": "2024-01-31",
      "status": "1"
    }
  }'
```

### 2. Create New User
```bash
curl -X POST https://your-domain.com/ost_wbs/ \
  -H "Content-Type: application/json" \
  -H "ApiKey: your-api-key-here" \
  -d '{
    "query": "user",
    "condition": "add",
    "parameters": {
      "name": "John Doe",
      "email": "john.doe@example.com",
      "phone": "1234567890",
      "org_id": "1",
      "default_email_id": "1",
      "status": "1",
      "timezone": "America/New_York",
      "password": "Securepassword123!"
    }
  }'
```

### 3. Create New Ticket
```bash
curl -X POST https://your-domain.com/ost_wbs/ \
  -H "Content-Type: application/json" \
  -H "ApiKey: your-api-key-here" \
  -d '{
    "query": "ticket",
    "condition": "add",
    "parameters": {
      "title": "Website Login Issue",
      "subject": "Cannot log into customer portal",
      "user_id": "123",
      "priority_id": "2",
      "status_id": "1",
      "dept_id": "1",
      "sla_id": "1",
      "topic_id": "1"
    }
  }'
```

### 4. Get Specific Ticket
```bash
curl -X GET https://your-domain.com/ost_wbs/ \
  -H "Content-Type: application/json" \
  -H "ApiKey: your-api-key-here" \
  -d '{
    "query": "ticket",
    "condition": "specific",
    "parameters": {
      "id": "123"
    }
  }'
```

### 5. List Departments
```bash
curl -X GET https://your-domain.com/ost_wbs/ \
  -H "Content-Type: application/json" \
  -H "ApiKey: your-api-key-here" \
  -d '{
    "query": "department",
    "condition": "all",
    "sort": "name"
  }'
```

### 6. Get Tasks for a Ticket
```bash
curl -X GET https://your-domain.com/ost_wbs/ \
  -H "Content-Type: application/json" \
  -H "ApiKey: your-api-key-here" \
  -d '{
    "query": "tasks",
    "condition": "all",
    "sort": "creationDate",
    "parameters": {
      "start_date": "2024-01-01",
      "end_date": "2024-01-31",
      "ticket_id": "123"
    }
  }'
```

## Error Handling

The API returns standardized error responses:

```json
{
  "status": "Error",
  "message": "Descriptive error message"
}
```

### Common Error Codes
- **API key not found/active**: Invalid or inactive API key
- **Source IP not authorized**: IP restriction enabled and request from unauthorized IP
- **Query not found**: Invalid entity name in query parameter
- **Condition not found**: Invalid method name for the specified entity
- **Empty or Incorrect fields**: Missing required parameters
- **Unexpected fields given**: Extra parameters not expected by the method
- **Invalid request method**: HTTP method not allowed for the operation

## Security Considerations

1. **Use HTTPS**: Always use HTTPS in production to protect API keys and data
2. **IP Restriction**: Enable `APIKEY_RESTRICT` for additional security
3. **API Key Management**: Regularly rotate API keys and limit permissions
4. **Input Validation**: The API automatically escapes parameters to prevent SQL injection
5. **Rate Limiting**: Consider implementing rate limiting at the web server level
6. **Firewall**: Restrict access to the API endpoint to trusted sources only

## System Requirements

- **PHP**: 7.0 or higher
- **MySQL**: 5.6 or higher (or compatible OSTicket database)
- **Extensions**: MySQLi, JSON
- **OSTicket**: Compatible with OSTicket v1.10+

## Contributing

We welcome contributions to improve this API! Here's how you can help:

### Development Setup
1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Make your changes and test thoroughly
4. Commit your changes: `git commit -m 'Add amazing feature'`
5. Push to the branch: `git push origin feature/amazing-feature`
6. Open a Pull Request

### Contribution Guidelines
- Follow PSR-12 coding standards
- Add appropriate comments for complex logic
- Test your changes with a real OSTicket installation
- Update documentation for new features
- Ensure backward compatibility when possible

### Reporting Issues
- Use the GitHub issue tracker
- Provide detailed reproduction steps
- Include PHP and OSTicket version information
- Sanitize any sensitive information from logs

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- **Original Developer**: Bruno Vieira
- **OSTicket Community**: For the amazing ticketing system
- **Contributors**: Everyone who has contributed to this project

## Additional Resources

- [Original Documentation](https://bmsvieira.gitbook.io/osticket-api/)
- [OSTicket Official Website](https://osticket.com/)
- [OSTicket Documentation](https://docs.osticket.com/)

---

<p align="center">
  <strong>Made with ❤️ for the OSTicket Community</strong><br>
  Feel free to contribute by opening a pull request!
</p>

