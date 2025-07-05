# MCP-UI SDK Architecture Research

## Overview

The **MCP-UI SDK** (Model Context Protocol UI SDK) is a TypeScript-based framework that enables interactive web components within the Model Context Protocol ecosystem. It allows MCP servers to deliver rich, dynamic UI resources that can be rendered by MCP-compliant clients/hosts.

## High-Level Architecture

### Core Concept
MCP-UI extends the Model Context Protocol by introducing UI resources that can be:
- Created on the server side
- Transmitted via standard MCP communication channels
- Rendered securely on the client side
- Interactive with bidirectional communication

### Repository Structure
```
mcp-ui/
├── packages/
│   ├── server/     # Server-side SDK for creating UI resources
│   ├── client/     # Client-side SDK for rendering UI resources
│   └── shared/     # Shared utilities and types
├── examples/
│   ├── server/     # Example MCP server implementation
│   └── remote-dom-demo/  # Remote DOM demo application
└── docs/          # Documentation
```

## Communication Flow

### 1. Server → Client Flow
```
MCP Server                    MCP Client/Host
    |                              |
    |-- Tool Call Response ------> |
    |   with UIResource            |
    |                              |
    |                          Parse Resource
    |                          Determine Type
    |                          Render Component
```

### 2. Client → Server Flow (UI Actions)
```
User Interaction              MCP Client/Host              MCP Server
    |                              |                           |
    |-- Click/Action -->           |                           |
    |                          Capture Event                   |
    |                          PostMessage                     |
    |                              |-- UI Action Result -->    |
    |                              |                           |
    |                              |<-- Tool Response ------   |
```

## Core Components

### Server-Side (`@mcp-ui/server`)

#### 1. **UIResource Creation**
- **Purpose**: Creates standardized UI resource payloads
- **Key Function**: `createUIResource(options)`
- **Resource Types**:
  - `rawHtml`: Inline HTML content
  - `externalUrl`: External URL to iframe
  - `remoteDom`: Remote DOM scripts

#### 2. **Resource Structure**
```typescript
interface UIResource {
  type: 'resource';
  resource: {
    uri: string;       // ui://component/id
    mimeType: string;  // text/html, text/uri-list, or remote-dom
    text?: string;     // Inline content
    blob?: string;     // Base64-encoded content
  };
}
```

#### 3. **UI Action Helpers**
- `uiActionResultToolCall()`: Trigger tool execution
- `uiActionResultPrompt()`: Send prompts
- `uiActionResultLink()`: Navigate to URLs
- `uiActionResultIntent()`: Custom intents
- `uiActionResultNotification()`: Show notifications

### Client-Side (`@mcp-ui/client`)

#### 1. **UIResourceRenderer**
- **Purpose**: Main component that determines resource type and renders appropriate sub-component
- **Supported Types**:
  - HTML resources (iframe-based)
  - External URLs (iframe-based)
  - Remote DOM (sandboxed JavaScript)

#### 2. **HTMLResourceRenderer**
- **Purpose**: Renders HTML content or external URLs in sandboxed iframes
- **Security**:
  - Uses `sandbox` attribute
  - Controlled PostMessage communication
  - Content isolation

#### 3. **RemoteDOMResourceRenderer**
- **Purpose**: Renders dynamic UI using Shopify's Remote DOM
- **Features**:
  - Sandboxed JavaScript execution
  - Component library mapping
  - React or Web Components rendering
- **Architecture**:
  ```
  Host Application
       |
  ThreadIframe (sandboxed)
       |
  Remote DOM Script Execution
       |
  JSON UI Description
       |
  Host Component Rendering
  ```

## Security Model

### 1. **Iframe Sandboxing**
All UI resources are rendered within sandboxed iframes:
- `allow-scripts`: Enables JavaScript execution
- `allow-same-origin`: For external URLs only
- No access to parent window except via PostMessage

### 2. **Message Validation**
- Origin verification for PostMessage events
- Typed action results
- Controlled communication channels

### 3. **Remote DOM Security**
- Scripts execute in isolated context
- UI updates transmitted as JSON
- No direct DOM access to host

## Component Libraries

### Basic Component Library
Provides fundamental UI components:
- **ui-text**: Text display
- **ui-button**: Interactive buttons
- **ui-stack**: Layout container
- **ui-image**: Image display

Each component maps remote attributes to React props and handles events appropriately.

## MCP Integration

### 1. **Server Integration**
```typescript
// Example from the server implementation
this.server.tool(
  'show_task_status',
  'Displays a UI for task status',
  async () => {
    const resourceBlock = createUIResource({
      uri: `ui://task-manager/${Date.now()}`,
      content: { type: 'externalUrl', iframeUrl: taskPageUrl },
      delivery: 'text',
    });
    return { content: [resourceBlock] };
  }
);
```

### 2. **Communication Protocols**
The example server supports multiple protocols:
- **HTTP Streaming**: `/mcp` endpoint
- **Server-Sent Events (SSE)**: `/sse` endpoint

### 3. **Tool Responses**
UI resources are returned as part of standard MCP tool responses, allowing seamless integration with existing MCP workflows.

## Key Design Patterns

### 1. **Resource Identification**
- URIs follow pattern: `ui://namespace/identifier`
- Unique URIs for caching and routing
- Timestamp-based IDs for uniqueness

### 2. **Delivery Modes**
- **text**: Direct string content
- **blob**: Base64-encoded content (for larger payloads)

### 3. **MIME Type Routing**
- `text/html`: Raw HTML content
- `text/uri-list`: External URLs
- `application/vnd.mcp-ui.remote-dom+javascript`: Remote DOM scripts

### 4. **Event Flow**
1. User interaction in iframe
2. PostMessage to parent
3. Parent captures and validates
4. Forwards to MCP server as needed
5. Server responds with appropriate action

## Extension Points

### 1. **Custom Component Libraries**
Developers can create custom component libraries by:
- Defining React components
- Mapping remote attributes to props
- Registering event handlers

### 2. **Resource Processors**
The `processResource` utility can be extended to support additional MIME types or resource formats.

### 3. **UI Action Types**
New action types can be added to support custom interactions beyond the built-in types.

## Best Practices for Contributors

### 1. **Resource Creation**
- Use unique URIs for each resource instance
- Choose appropriate delivery mode (text vs blob)
- Include proper MIME types

### 2. **Security Considerations**
- Always validate message origins
- Sanitize user inputs
- Use minimal sandbox permissions

### 3. **Performance**
- Prefer text delivery for small content
- Use blob encoding for large HTML
- Cache resources when possible

### 4. **Component Development**
- Keep components stateless when possible
- Handle both controlled and uncontrolled modes
- Provide proper TypeScript types

## Future Considerations

Based on the roadmap, the project is moving towards:
- Additional frontend framework support
- Expanded component libraries
- Declarative UI content types
- Potential generative UI capabilities

## Summary

MCP-UI provides a secure, flexible framework for embedding interactive UI components within MCP workflows. Its architecture emphasizes:
- **Security** through sandboxing and controlled communication
- **Flexibility** with multiple resource types and rendering modes
- **Extensibility** via component libraries and custom actions
- **Integration** with standard MCP protocols and patterns

This makes it an ideal solution for enhancing MCP servers with rich user interfaces while maintaining the security and simplicity of the Model Context Protocol.