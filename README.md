# Building a company enrichment agent using Apollo MCP with Tasks API

Parallel Tasks allow [MCP tool calling](https://docs.parallel.ai/features/mcp-tool-call) as part of their Task API (now in Beta) which is very exciting! Besides the task using web data this allows you to use any private data. In this guide I'm going to show you how this works.

First, let's see what the AI needs to know to build something with Tasks with MCP tools; appending `.md` to the docs pages give me the context I need in markdown format:

- https://docs.parallel.ai/features/mcp-tool-call.md
- https://docs.parallel.ai/task-api/core-concepts/specify-a-task.md
- https://docs.parallel.ai/api-reference/task-api-v1/create-task-run.md
- https://docs.parallel.ai/api-reference/task-api-v1/retrieve-task-run-result.md

Let's [ask the AI](https://letmeprompt.com/rules-httpsuithu-k0w66o0) to make a nice curl to see if this works. MCP used: https://docs.devin.ai/work-with-devin/deepwiki-mcp.md (url: https://mcp.deepwiki.com/mcp). My key learning here is that you cannot ask the task about the tools itself, so I slightly altered the task to use one of the actual tools, which worked brilliantly!

```sh
API_KEY="YOUR_KEY" && \
RUN_ID=$(curl -s -X POST "https://api.parallel.ai/v1/tasks/runs" \
  -H "x-api-key: $API_KEY" \
  -H "Content-Type: application/json" \
  -H "parallel-beta: mcp-server-2025-07-17" \
  --data '{
    "input": "What is the purpose of janwilmake/dorm and how to use it?",
    "processor": "lite",
    "mcp_servers": [
      {
        "type": "url",
        "url": "https://mcp.deepwiki.com/mcp",
        "name": "deepwiki_mcp"
      }
    ]
  }' | tee /dev/stderr | jq -r '.run_id') && \
echo "Run ID: $RUN_ID" && \
curl -s "https://api.parallel.ai/v1/tasks/runs/$RUN_ID/result" \
  -H "x-api-key: $API_KEY" | tee /dev/stderr | jq '.'
```

Result:

```json
{
  "run": {
    "run_id": "trun_36d27f69cc5d41b4bfbf17f12a642cc6",
    "status": "completed",
    "is_active": false,
    "warnings": null,
    "error": null,
    "processor": "lite",
    "metadata": {},
    "taskgroup_id": null,
    "created_at": "2025-08-26T10:47:05.317955Z",
    "modified_at": "2025-08-26T10:47:27.821851Z"
  },
  "output": {
    "basis": [
      {
        "field": "output",
        "citations": [],
        "reasoning": "The response is based on the information from the deepwiki_mcp tool call, which provided a detailed explanation of the janwilmake/dorm repository's purpose and usage.",
        "confidence": ""
      }
    ],
    "mcp_tool_calls": [
      // 2 tool calls
    ],
    "type": "json",
    "content": {
      "output": "The `janwilmake/dorm` repository offers DORM (Durable Object Relational Mapping), a system that simplifies interactions with SQLite databases within Cloudflare Durable Objects. It abstracts the complexities of managing Durable Objects, providing a developer-friendly interface for SQL operations on distributed SQLite databases. To use DORM, you typically interact with the `createClient` function. First, install DORM using npm: `npm i dormroom`. Then, initialize a DORM client by calling `createClient`, passing your `DurableObjectNamespace`, a `DBConfig` object, and optional `doConfig`. Once the client is initialized, you can perform various database operations like raw SQL queries or ORM operations."
    },
    "output_schema": {}
  }
}
```

# Using Authorized MCPs

Great! Now, let's see if we can also use MCP servers that require authorization. This should be possible by passing authorization header to the MCP tool.

# Using Typescript SDK

Context:

- https://docs.parallel.ai/features/mcp-tool-call.md
- https://raw.githubusercontent.com/parallel-web/parallel-cookbook/refs/heads/main/typescript-sdk-types.d.ts (needs newer version)

TODO:

- First, try it with SDK with https://mcp.deepwiki.com/mcp MCP
- If that succeeds, use Apollo MCP - https://github.com/janwilmake/apollo-io-mcp-server

Learnings:

- You can not ask it about the available tools
- Unlike the docs suggest, even with lite, it's possible that the task performs more than one tool call
