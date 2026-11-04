# Milestone 3 · End of Week 6, start of Week 7 · The MCP server

**Time:** about 3 evenings.
**New idea:** MCP (Model Context Protocol) is a standard plug for tools. You write your tools once, and any MCP app can use them: Claude Desktop, your own agent, or any other MCP client.

[← Back to the project overview](<README.md>)

---

## The idea

```
 Before:  your agent ──imports──▶ tools.ts

 After:   your agent ─────┐
          Claude Desktop ─┼── MCP (JSON-RPC over stdio) ──▶ your MCP server ──▶ execute() ──▶ tools.ts
          any MCP client ─┘
```

Four things to understand:

1. **Two requests do most of the work.** `tools/list` ("what can you do?") and `tools/call` ("do this, with these arguments"). The SDK handles the rest.
2. **With stdio, stdout is the wire.** The client reads your server's standard output as protocol messages. One stray `console.log` puts plain text where the client expects JSON, and the connection breaks. Log with `console.error`, which goes to stderr.
3. **Descriptions are now read by strangers.** In Milestone 2, your own agent read your tool descriptions. Now any model in any MCP app reads them, with no other context. This week's Day 3 (tool schema design) is about exactly this.
4. **One copy of the tool code.** The server doesn't copy your tools. It imports them, and calls the same `execute()` from Milestone 2, so every client gets the same Zod checks.

**Why not a normal HTTP API?** MCP clients find your tools by themselves and know how to hand them to a model. An HTTP API needs custom glue code in every app that wants to use it.

---

## Requirements

| ID | Requirement |
|---|---|
| M1 | An MCP server in TypeScript (`@modelcontextprotocol/sdk`, stdio transport) that exposes the five tools from Milestone 2. The tool code is **shared**, not copied: the server imports `tools` and `execute()`. |
| M2 | Each tool has a description a stranger could use: what it does, when to use it, what it returns, and its limits. Rewrite your Milestone 2 descriptions. |
| M3 | It works from **two** clients: Claude Desktop (or another MCP client), and your own agent (M4). Screenshot Claude Desktop calling one of your tools. |
| M4 | Change your Milestone 2 agent so it gets its tools from the MCP server, using the SDK's `Client` and `StdioClientTransport`, instead of importing them. |
| M5 | Your server never writes to stdout except through the MCP transport. Logs go to stderr (`console.error`). |

## Steps

1. **Install** the SDK: `pnpm add @modelcontextprotocol/sdk@1.32.2`.
2. **The server** (`src/mcp/server.ts`). Follow the Week 6 Day 2 lab's pattern:
   - create a `Server` with the `tools` capability;
   - handle `tools/list` by mapping your tools to `{ name, description, inputSchema }`, with `inputSchema` from `z.toJSONSchema(schema)`;
   - handle `tools/call` by passing the name and arguments to your `execute()`. Return `{ content: [{ type: "text", text }] }`, and set `isError: true` when `execute()` returned an error, so the model knows the call failed;
   - connect a `StdioServerTransport`.
3. **Test it with the MCP Inspector:** `npx @modelcontextprotocol/inspector npx tsx src/mcp/server.ts`. List the tools, and call `list_emails` by hand. Then call `get_recent_commits` with `{"days": 30}`: you should get a readable error, and the server should keep running.
4. **Break it on purpose.** Add a `console.log("hello")` at the top of the server and reconnect the Inspector. See what happens. Change it to `console.error`.
5. **Claude Desktop.** Build with `pnpm build`, then add your server to Claude Desktop's config file (on Windows: `%APPDATA%\Claude\claude_desktop_config.json`):

   ```json
   {
     "mcpServers": {
       "career-copilot": {
         "command": "node",
         "args": ["C:\\full\\path\\to\\career-copilot\\dist\\mcp\\server.js"],
         "env": {
           "OPENAI_API_KEY": "your key",
           "GITHUB_USER": "your username",
           "GITHUB_REPOS": "repo-one,repo-two",
           "DATA_DIR": "C:\\full\\path\\to\\career-copilot\\data"
         }
       }
     }
   }
   ```

   Restart Claude Desktop and ask it "anything from recruiters in my inbox?"
6. **Your agent as a client.** In your loop, start the server with `StdioClientTransport`, get the tool list with `client.listTools()`, pass it to OpenAI, and run each tool call with `client.callTool({ name, arguments })`.

## Hints

- **"It works in the terminal but not in Claude Desktop."** Claude Desktop starts your server from a different folder, so `data/emails.json` isn't found. Read the data folder from a `DATA_DIR` environment variable, with `data` as the default.
- **A file that runs code when imported.** If importing `execute()` also starts an agent or creates an OpenAI client at the top level, the server does that too. Keep `execute.ts` free of side effects.
- **Claude Desktop's config file holds your key.** It lives outside your repo. Never copy it in.
- **Windows trouble with stdio?** Use the MCP Inspector as your second client. It's the same lesson with fewer config problems.

## Done when

- [ ] The Inspector lists all five tools, and a bad argument returns a readable error without stopping the server.
- [ ] Claude Desktop (or another MCP client) answers a question by calling one of your tools. Screenshot it.
- [ ] Your agent gets its tools through MCP, and gives the same answers as in Milestone 2.
- [ ] `grep -rn "api.github.com" src` finds exactly one place.
- [ ] `grep -rn "console.log" src/mcp src/agent/tools.ts src/agent/execute.ts` finds nothing.
- [ ] Your README explains in three sentences, in your own words, why MCP is useful, and when you wouldn't use it.

## Stretch

- **Use it from n8n.** n8n's MCP Client Tool node connects over HTTP, not stdio, and your n8n runs inside Docker. Add the SDK's Streamable HTTP transport on a port, then point n8n at `http://host.docker.internal:<port>/mcp`.
- **Add a resource.** Expose your recent posts as an MCP resource (for example, `posts://recent`). Explain the difference: a resource is data the client reads; a tool is an action the model asks for.

---

## Hand in

Push to your repo, with the Claude Desktop screenshot and your three-sentence MCP explanation in the README.

**Next:** [Milestone 4 · The web app](<milestone-4-web-app.md>).
