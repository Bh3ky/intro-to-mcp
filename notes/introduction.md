## Introducing MCP

- Model Context Protocol (MCP) is a communication layer that provides Claude with context and tools without requiring us to write a bunch of tedious integration code.
- often, we see diagrams depicting the basic architecture of an MCP Client (which is our server) connecting to MCP Servers that contain tools, prompt, and resources.
- each MCP Server acts as an interface to some outside service.

- to have a complete fully functional GitHub chatbot we use MCP as it shifts the burden of moving tool definitions and executions from our server to the dedicated MCP servers.
  - it wraps up tons of functionality and exposes it as a standardised set of tools.

### MCP Servers Explained
- MCP Servers provide access to data or functionality implemented by outside services.
  - they act as specialised interfaces that expose tools, prompts, and resources in a standardised way. 
- e.g., for our GitHub chatbot, the MCP Server for GitHub would contain tools like `get_repos` and connects directly to GitHub's API.
- the server would communicate with the MCP Server which handles all the GitHub-specific implementation details.

> MCP Server basically wraps up access to some service.

- MCP Servers provide tool schemas + functions already defined for us.

---

## MCP clients

- MCP client serves as the communication bridge between our server and MCP servers.
- it's the access point to all the tools that an MCP server provides, handling the message exchange and protocol details so our application doesn't have to. 

### Transport Agnostic Communication

- one of MCP's key strengths is being transport agnostic i.e., the client and server can communicate over different protocols depending on our setup.
- most common setups:
  - we can run both MCP client and server on the same machine communicating through standard I/O.
  - HTTP, WebSockets, and various other network protocols.

## MCP Message Types

- client and server can exchange specific message types defined in the MCP specification.
- for example:
  - ListToolsRequest/ListToolsResult i.e., client can ask the server what tools it provides & they can get back a list of available tools.
  - CallToolRequest/CallToolResult i.e., client asks the server to run a specific tool with given arguments,then receives the result.

HOW IT ALL WORKS TOGETHER??

- here is a complete example showing how a user query flows through the entire system - from our server, through the MCP client, to external services like GitHub, and back to Claude.
- step-by-step flow:
    1. User Query: the user submits their questions to our server
    2. Tool Discovery: our server needs to know what tools are available to send to Claude
    3. ListToolsExchange: our server asks the MCP client for available tools
    4. MCP Communication: the MCP client sends a `ListToolsRequest` to the MCP server and receives a `ListToolsResult`
    5. Claude Request: our server sends the user's query plus the available tools to Claude
    6. Tool Use Decision: Claude decides it needs to call a tool to answer the question
    7. Tool Execution Request: Our server asks the MCP client to run the tool Claude specified
    8. External API Call: The MCP client sends a `CallToolRequest` to the MCP server, which makes the actual GitHub API call
    9. Results Flow Back: GitHub responds with repository data, which flows back through the MCP server as a `CallToolResult`
    10. Tool Result to Claude: Our server sends the tool results back to Claude
    11. Final Response: Claude formulates a final answer using the repository data
    12. User Gets Answer: Your server delivers Claude's response back to the user

- the MCP client abstracts away the complexity of server communication, letting us focus on our application logic while still getting access to powerful external tools and data sources.
