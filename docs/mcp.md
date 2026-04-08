# MCP (Model Context Protocol)

The `@dashnex/mcp` module exposes an MCP server over HTTP using JSON-RPC 2.0. AI agents can call tools, read resources, and use prompts registered by any module.

## Endpoint

```
POST /api/mcp
```

## Request Flow

```
POST /api/mcp → Extract user → Parse JSON-RPC 2.0 → Route by method:
  ├─ initialize       → server capabilities
  ├─ tools/list       → emit mcp.tools_get → return tools
  ├─ tools/call       → emit mcp.tools_get → find handler → execute
  ├─ resources/list   → emit mcp.resources_get → return resources
  ├─ resources/read   → emit mcp.resources_get → find handler → execute
  ├─ prompts/list     → emit mcp.prompts_get → return prompts
  └─ prompts/get      → emit mcp.prompts_get → find handler → execute
```

## Registering Tools

Listen to `mcp.tools_get`:

```ts
import type { McpToolsGetContext } from '@dashnex/mcp'
import { hasRole } from '@dashnex/auth'

export async function onMcpToolsGet(context: McpToolsGetContext): Promise<void> {
  if (!context.user || !hasRole(context.user, 'admin')) return

  context.tools.push({
    name: 'my_module_do_something',
    description: 'Does something useful',
    inputSchema: {
      type: 'object',
      properties: {
        id: { type: 'string', description: 'Resource ID' },
      },
      required: ['id'],
    },
  })

  context.toolHandlers.set('my_module_do_something', async (args) => {
    const service = await getMyService()
    const result = await service.doSomething(args.id as string)
    return {
      content: [{ type: 'text', text: JSON.stringify({ success: true, result }) }]
    }
  })
}
```

Register in module config:

```ts
listeners: [
  { event: 'mcp.tools_get', handler: onMcpToolsGet },
]
```

## Registering Resources

```ts
import type { McpResourcesGetContext } from '@dashnex/mcp'

export async function onMcpResourcesGet(context: McpResourcesGetContext): Promise<void> {
  if (!context.user) return

  context.resources.push({
    uri: 'myapp://docs/readme',
    name: 'Readme',
    description: 'Application readme',
    mimeType: 'text/markdown',
  })

  context.resourceHandlers.set('myapp://docs/readme', async () => ({
    contents: [{ uri: 'myapp://docs/readme', mimeType: 'text/markdown', text: '# Hello' }]
  }))
}
```

## Registering Prompts

```ts
import type { McpPromptsGetContext } from '@dashnex/mcp'

export async function onMcpPromptsGet(context: McpPromptsGetContext): Promise<void> {
  if (!context.user) return

  context.prompts.push({
    name: 'greet',
    description: 'Generate a greeting',
    arguments: [{ name: 'name', description: 'User name', required: true }],
  })

  context.promptHandlers.set('greet', async (args) => ({
    messages: [{ role: 'user', content: { type: 'text', text: `Say hello to ${args.name}` } }]
  }))
}
```

## Organizing Tools by Role

```ts
export async function onMcpToolsGet(context: McpToolsGetContext): Promise<void> {
  if (!context.user) return

  if (hasRole(context.user, 'admin')) {
    registerAdminTools(context)
  }

  const contact = await contactService.getContactByUser(context.user)
  if (contact) {
    registerUserTools(context, { contact })
  }
}
```

## Types

```ts
import type {
  McpToolsGetContext,
  McpResourcesGetContext,
  McpPromptsGetContext,
  McpTool,
  McpResource,
  McpPrompt,
} from '@dashnex/mcp'
```
