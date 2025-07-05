# MCP-UI Repository Research Summary

## Executive Summary

MCP-UI is a TypeScript SDK that extends the Model Context Protocol (MCP) by enabling MCP servers to create and serve interactive UI components. It consists of two main packages:
- **@mcp-ui/server**: For creating UI resources on MCP servers
- **@mcp-ui/client**: For rendering UI resources in MCP clients/hosts

## Key Findings

### 1. Architecture Overview
- **Monorepo Structure**: Uses pnpm workspaces with separate packages for server, client, and shared code
- **Security-First Design**: All UI content runs in sandboxed iframes with controlled communication
- **Three UI Types**: Raw HTML, External URLs, and Remote DOM (dynamic JavaScript-based UI)

### 2. Communication Model
```
MCP Server → UIResource → MCP Client → Sandboxed Iframe → PostMessage → UI Actions
```

- Server creates resources using `createUIResource()`
- Client renders using `<UIResourceRenderer />`
- User interactions trigger PostMessage events
- Events can trigger MCP tool calls, prompts, or other actions

### 3. Core Technologies
- **TypeScript**: Full type safety across boundaries
- **React**: For client-side rendering
- **Shopify Remote DOM**: For secure dynamic UI
- **PostMessage API**: For cross-origin communication
- **MCP SDK**: For protocol integration

### 4. Security Features
- Iframe sandboxing with minimal permissions
- Message source validation
- No direct DOM access from remote code
- JSON-based UI descriptions for Remote DOM

### 5. Current Limitations
- Only basic component library available
- React-only for Remote DOM rendering
- Limited testing infrastructure
- No declarative UI format yet

## How It Works

### Server Side
1. Import `@mcp-ui/server`
2. Create UI resources with specific MIME types
3. Return resources in MCP tool responses

### Client Side
1. Import `@mcp-ui/client`
2. Detect UI resources in MCP responses
3. Render with `UIResourceRenderer`
4. Handle UI actions via callbacks

### Example Flow
```typescript
// Server
const resource = createUIResource({
  uri: 'ui://demo/123',
  content: { type: 'rawHtml', htmlString: '<button>Click</button>' },
  delivery: 'text'
});

// Client
<UIResourceRenderer 
  resource={resource}
  onUIAction={(action) => handleAction(action)}
/>
```

## Contribution Opportunities

### High-Impact Areas
1. **Component Libraries**: Material UI, Tailwind, Bootstrap implementations
2. **Framework Support**: Vue, Svelte, Angular adapters
3. **Developer Tools**: Resource preview, visual builders
4. **Testing**: E2E tests, visual regression tests
5. **Documentation**: Tutorials, examples, best practices

### Quick Wins
- Add more components to basic library
- Improve error handling and messages
- Add TypeScript type generators
- Create example applications
- Write integration guides

## Technical Insights

### Strengths
- Clean separation of concerns
- Extensible architecture
- Strong security model
- Good TypeScript support
- Active development

### Areas for Improvement
- More comprehensive testing
- Better developer experience tools
- Performance optimizations
- Accessibility features
- Cross-framework support

## Getting Started as a Contributor

1. **Understand the Flow**: Server creates → Client renders → Actions handled
2. **Start Small**: Add a component or fix a bug
3. **Test Everything**: Both unit and integration tests
4. **Follow Patterns**: Study existing code for conventions
5. **Ask Questions**: Use GitHub discussions for clarification

## Key Takeaways

MCP-UI successfully extends MCP with UI capabilities while maintaining security through:
- Sandboxed execution environments
- Typed communication protocols
- Flexible resource types
- Extensible component system

The project is well-positioned for growth with clear extension points and a solid foundation. Contributors can make significant impact by enhancing component libraries, adding framework support, or improving developer experience.

---

This research provides a comprehensive understanding of MCP-UI's architecture, implementation, and contribution opportunities. The SDK offers a unique approach to adding UI capabilities to MCP servers while maintaining the protocol's security and simplicity principles.