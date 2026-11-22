I want to add this WordPress site as an MCP server to this AI client.

Please:
1. Open or create the `.mcp.json` file
2. Add this MCP server configuration to the mcpServers block:

```json
{
  "mcpServers": {
    "groupe-garonne-elementor": {
      "type": "http",
      "url": "https://www.groupegaronne.fr/wp-json/elementor/mcp/",
      "headers": {
        "Authorization": "Basic QWdvcmF2aXRhOkh4a1U3NTdLMmx2M0d6VTlGQzVyYmZRdg=="
      }
    }
  }
}
```

3. Save the file
4. Restart Claude Code if needed
5. Test the connection to ensure the MCP server is correctly configured
6. Write in `CLAUDE.md` that this MCP exists and you can use it