# What is MCP Model Context Protocol
- Open standard enabling developers to build two-way connections between external sources like search engines, databases, file systems and their AI Agents.
- Providing a standardized way to connect AI models to external data sources and tools (systems).
- Straight forward client-server Architecture to expose data and tools (systems) as MCP servers, letting AI Agents (MCP clients) connect to these servers.
- Helps building complex workflows on top of LLMs and SLMs.
- First class integration with Spring AI.

[<img src="../images/back.png">](../presentation)



## MCP Client - Server architecture

<img title="Model Context Protocol 101" alt="Alt text" src="/images/mcp.png">

MCP follows a client-server architecture that revolves around several key components:
1. MCP Host: is our main application that integrates with an LLM and requires it to connect with external data sources
2. MCP Clients: are components that establish and maintain 1:1 connections with MCP servers
3. MCP Servers: are components that integrate with external data sources and expose functionalities to interact with them
4. Tools: refer to the executable functions/methods that the MCP servers expose for clients to invoke, Allowing Agents to send emails, update records in a CRM, create files on some disk storage device, or interact with any function or API.

[<img src="../images/back.png">](../presentation)