## Using HTTP Headers for Authentication

When running the MCP server in HTTP transport mode, you can provide Alpaca API credentials via HTTP headers instead of environment variables. This is particularly useful for:

- Remote API access without exposing credentials in environment files
- Multiple users accessing the same MCP server with different credentials
- Integrating with existing authentication systems

### Starting the Server in HTTP Mode

```bash
python alpaca_mcp_server.py --transport http --host 0.0.0.0 --port 8000
```

### Making API Requests with Credentials in Headers

You can provide your Alpaca API credentials in the HTTP headers of your requests using a Bearer token:

```bash
curl -X POST \
  http://localhost:8000/mcp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ALPACA_API_KEY:YOUR_ALPACA_SECRET_KEY" \
  -d '{"mcp": {"jsonrpc": "2.0", "id": 1, "method": "get_account_info"}}'
```

### Header Details

| Header Name | Format | Description |
|-------------|--------|-------------|
| `Authorization` | `Bearer {api_key}:{api_secret}` | Standard Bearer token format containing your Alpaca API key and secret separated by a colon |

### How It Works

This implementation uses FastMCP's `get_context()` function to access the request context and headers directly within each client getter function. When a tool needs an Alpaca client:

1. The tool calls a client getter function (e.g., `get_trading_client()`) 
2. The getter function checks if it's running in HTTP mode by examining the context
3. If in HTTP mode, it looks for credentials in the Authorization header
4. If a valid Bearer token is found, it parses the API key and secret from the token
5. If credentials are successfully extracted, it returns a new client instance using those credentials
6. If no credentials are found or not running in HTTP mode, it falls back to environment variables

### Fallback Behavior

If you don't provide credentials in the headers, the server will fall back to using the credentials from environment variables (if available). This allows for flexible deployment where some requests might include credentials and others might use the default credentials.

### Security Considerations

- When exposing the HTTP interface publicly, always use HTTPS
- Consider implementing additional authentication for the MCP server itself
- The headers use the same names as the official Alpaca API for consistency
- Each request gets its own client instance with the provided credentials
- Credentials are never stored globally and are scoped to the specific request
- This per-request client model ensures complete isolation between different users' requests