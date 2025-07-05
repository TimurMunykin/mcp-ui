# MCP-UI Contributor Guide

## Quick Start for Contributors

### What is MCP-UI?

MCP-UI is a TypeScript SDK that extends the Model Context Protocol (MCP) with interactive UI capabilities. It allows MCP servers to create and serve rich UI components that can be rendered securely by MCP clients.

### Key Concepts You Need to Know

1. **MCP (Model Context Protocol)**: A protocol for communication between AI agents and tools/servers
2. **UI Resources**: Special MCP resources that contain UI content (HTML, URLs, or Remote DOM scripts)
3. **Sandboxed Rendering**: All UI content runs in isolated iframes for security
4. **PostMessage Communication**: How UI components communicate with the host application

## Development Setup

### Prerequisites
- Node.js 18+
- pnpm 8.15.7+
- Git

### Initial Setup
```bash
# Clone the repository
git clone https://github.com/idosal/mcp-ui.git
cd mcp-ui

# Install dependencies
pnpm install

# Build all packages
pnpm build
```

### Repository Structure
```
mcp-ui/
├── packages/
│   ├── server/     # npm: @mcp-ui/server
│   ├── client/     # npm: @mcp-ui/client
│   └── shared/     # Internal shared utilities
├── examples/
│   ├── server/     # Example MCP server (deployed to Cloudflare)
│   └── remote-dom-demo/  # Local testing app
```

## Core Architecture

### Communication Flow
```
1. MCP Server creates UIResource via createUIResource()
2. Resource sent in tool response
3. Client receives and detects resource type
4. UIResourceRenderer routes to appropriate renderer
5. Content renders in sandboxed iframe
6. User interactions trigger PostMessage events
7. Events forwarded to MCP server as needed
```

### Three Types of UI Resources

1. **Raw HTML** (`text/html`)
   - Inline HTML rendered via iframe srcDoc
   - Good for simple, self-contained UI

2. **External URL** (`text/uri-list`)
   - External web app in iframe
   - Good for complex existing applications

3. **Remote DOM** (`application/vnd.mcp-ui.remote-dom`)
   - JavaScript that generates UI dynamically
   - UI description sent as JSON to host
   - Rendered with host's component library
   - Good for native-feeling UI

## Key Areas for Contribution

### 1. Component Libraries
Current state: Only basic component library exists

Opportunities:
- Create Material UI component library
- Create Tailwind UI component library
- Add more components to basic library
- Create themed component sets

Example structure:
```typescript
export const myComponentLibrary: ComponentLibrary = {
  name: 'material-ui',
  elements: [
    {
      tagName: 'mui-button',
      component: MuiButton,
      propMapping: { label: 'children', variant: 'variant' },
      eventMapping: { press: 'onClick' }
    }
  ]
};
```

### 2. Framework Support
Current state: React-only for Remote DOM

Opportunities:
- Add Vue.js support
- Add Svelte support
- Add Web Components enhancements
- Create framework adapters

### 3. Security Enhancements
Current state: Basic iframe sandboxing

Opportunities:
- Add CSP (Content Security Policy) support
- Implement origin validation
- Add resource signing/verification
- Create security audit tools

### 4. Developer Experience
Current state: Basic SDK functionality

Opportunities:
- Create UI resource preview tool
- Add TypeScript generators for Remote DOM
- Create visual component builder
- Add development mode with hot reload

### 5. Testing Infrastructure
Current state: Basic unit tests

Opportunities:
- Add E2E testing framework
- Create visual regression tests
- Add performance benchmarks
- Create testing utilities for consumers

## Common Development Tasks

### Adding a New Resource Type
1. Update server types in `packages/server/src/types.ts`
2. Add handling in `createUIResource()` function
3. Update client types in `packages/client/src/types.ts`
4. Add detection logic in `UIResourceRenderer`
5. Create new renderer component
6. Add tests for both server and client

### Creating a Component Library
1. Create components in `packages/client/src/remote-dom/component-libraries/`
2. Define component mappings (props and events)
3. Export as ComponentLibrary type
4. Add examples in documentation
5. Create tests for component behavior

### Testing Your Changes
```bash
# Run tests for all packages
pnpm test

# Run tests in watch mode
pnpm test:watch

# Run specific package tests
cd packages/server && pnpm test

# Build and test examples
cd examples/server && pnpm build
```

## Important Patterns

### Server-Side Pattern
```typescript
// Always use unique URIs
const uri = `ui://feature/${Date.now()}`;

// Choose appropriate delivery
const delivery = content.length > 100000 ? 'blob' : 'text';

// Return in standard format
return {
  content: [createUIResource({ uri, content, delivery })]
};
```

### Client-Side Pattern
```typescript
// Always validate resource type
if (!resource.uri?.startsWith('ui://')) return null;

// Handle errors gracefully
try {
  return <UIResourceRenderer resource={resource} />;
} catch (error) {
  console.error('Failed to render:', error);
  return <ErrorFallback />;
}
```

### Security Pattern
```typescript
// Always validate message source
if (event.source !== iframe.contentWindow) return;

// Type check action results
if (!isValidUIAction(event.data)) return;

// Handle errors in handlers
onUIAction?.(event.data).catch(console.error);
```

## Debugging Tips

1. **Use the MCP Inspector**: The example server can be tested with MCP inspector tools
2. **Check iframe sandbox**: Ensure proper permissions for your use case
3. **Monitor PostMessage**: Use browser DevTools to see message flow
4. **Test resource encoding**: Verify base64 encoding/decoding for blobs
5. **Validate MIME types**: Ensure correct MIME type for resource type

## Getting Help

- **GitHub Issues**: For bugs and feature requests
- **Discussions**: For questions and ideas
- **Examples**: Study the example implementations
- **Tests**: Read tests to understand expected behavior

## Before Submitting PR

1. Run all tests: `pnpm test`
2. Format code: `pnpm prettier --write .`
3. Update documentation if needed
4. Add tests for new features
5. Follow commit message conventions
6. Ensure examples still work

## Architectural Decisions

- **Why iframes?** Security isolation for untrusted content
- **Why Remote DOM?** Native UI feel without sacrificing security
- **Why PostMessage?** Standard, secure cross-origin communication
- **Why TypeScript?** Type safety across server/client boundary

## Future Roadmap Areas

Based on the project roadmap, consider contributing to:
- Declarative UI content types (JSON-based UI definitions)
- Generative UI capabilities (AI-generated interfaces)
- Enhanced component libraries
- Performance optimizations
- Accessibility improvements

Remember: The goal is to make MCP servers capable of rich interactions while maintaining security and simplicity!