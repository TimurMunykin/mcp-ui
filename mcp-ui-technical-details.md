# MCP-UI Technical Implementation Details

## Server-Side Implementation

### Creating UI Resources

The server package provides a simple API for creating UI resources:

```typescript
import { createUIResource } from '@mcp-ui/server';

// Example 1: Raw HTML Resource
const htmlResource = createUIResource({
  uri: 'ui://my-component/123',
  content: { 
    type: 'rawHtml', 
    htmlString: '<div>Hello <button onclick="parent.postMessage({type: \'tool\', payload: {toolName: \'greet\', params: {}}}, \'*\')">Click me</button></div>' 
  },
  delivery: 'text'
});

// Example 2: External URL Resource
const urlResource = createUIResource({
  uri: 'ui://external-app/456',
  content: { 
    type: 'externalUrl', 
    iframeUrl: 'https://myapp.com/widget' 
  },
  delivery: 'text'
});

// Example 3: Remote DOM Resource (React)
const remoteDomResource = createUIResource({
  uri: 'ui://remote-component/789',
  content: {
    type: 'remoteDom',
    flavor: 'react',
    script: `
      const button = document.createElement('ui-button');
      button.setAttribute('label', 'Click me');
      button.addEventListener('press', () => {
        window.parent.postMessage({
          type: 'tool',
          payload: { toolName: 'buttonClicked', params: {} }
        }, '*');
      });
      root.appendChild(button);
    `
  },
  delivery: 'text'
});
```

### MCP Tool Integration

UI resources are typically returned from MCP tool handlers:

```typescript
server.tool(
  'show_dashboard',
  'Shows an interactive dashboard',
  async () => {
    const resource = createUIResource({
      uri: `ui://dashboard/${Date.now()}`,
      content: { type: 'externalUrl', iframeUrl: 'https://myapp.com/dashboard' },
      delivery: 'text'
    });
    
    return {
      content: [resource]  // Returns as part of tool response
    };
  }
);
```

## Client-Side Implementation

### Resource Rendering Flow

```typescript
// 1. Main Renderer Component
<UIResourceRenderer 
  resource={mcpResource.resource}
  onUIAction={handleUIAction}
  supportedContentTypes={['rawHtml', 'externalUrl', 'remoteDom']}
/>

// 2. Resource Type Detection
function getContentType(resource: Partial<Resource>): ResourceContentType | undefined {
  if (resource.mimeType === 'text/html') return 'rawHtml';
  if (resource.mimeType === 'text/uri-list') return 'externalUrl';
  if (resource.mimeType?.startsWith('application/vnd.mcp-ui.remote-dom')) return 'remoteDom';
}

// 3. Routing to Specific Renderer
switch (contentType) {
  case 'rawHtml':
  case 'externalUrl':
    return <HTMLResourceRenderer ... />;
  case 'remoteDom':
    return <RemoteDOMResourceRenderer ... />;
}
```

### HTML Resource Rendering

The HTMLResourceRenderer handles both inline HTML and external URLs:

```typescript
// For inline HTML (srcDoc mode)
<iframe
  srcDoc={htmlString}
  sandbox="allow-scripts"
  style={{ width: '100%', minHeight: 200 }}
/>

// For external URLs (src mode)
<iframe
  src={iframeSrc}
  sandbox="allow-scripts allow-same-origin"
  style={{ width: '100%', minHeight: 200 }}
/>
```

### Remote DOM Implementation

Remote DOM uses a more complex architecture with thread-based communication:

```typescript
// 1. Create sandboxed iframe with bundled Remote DOM runtime
<iframe
  srcDoc={IFRAME_SRC_DOC}  // Contains Remote DOM runtime
  sandbox="allow-scripts"
  onLoad={handleIframeLoad}
/>

// 2. Establish thread communication
const thread = new ThreadIframe<SandboxAPI>(iframe);

// 3. Execute remote script in sandbox
thread.imports.render({
  code: resource.content,
  remoteElements: remoteElementsConfig,
  useReactRenderer: true,
  componentLibrary: 'basic'
}, receiver.connection);

// 4. Render received UI updates
<RemoteRootRenderer 
  receiver={receiver} 
  components={componentMap} 
/>
```

## Message Communication

### UI Action Flow

1. **User Interaction** → 2. **PostMessage** → 3. **Client Handler** → 4. **MCP Server**

```typescript
// In iframe/UI resource:
window.parent.postMessage({
  type: 'tool',
  payload: {
    toolName: 'submitForm',
    params: { name: 'John', email: 'john@example.com' }
  }
}, '*');

// In client handler:
function handleMessage(event: MessageEvent) {
  if (event.source === iframe.contentWindow) {
    const action = event.data as UIActionResult;
    onUIAction?.(action);  // Forward to MCP server
  }
}
```

### Supported Action Types

```typescript
type UIActionResult =
  | { type: 'tool'; payload: { toolName: string; params: Record<string, unknown> } }
  | { type: 'prompt'; payload: { prompt: string } }
  | { type: 'link'; payload: { url: string } }
  | { type: 'intent'; payload: { intent: string; params: Record<string, unknown> } }
  | { type: 'notification'; payload: { message: string } };
```

## Component Library System

### Defining Components

```typescript
const UIButton = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ label, onPress, children, ...props }, ref) => {
    return (
      <button ref={ref} onClick={onPress} {...props}>
        {label || children}
      </button>
    );
  }
);

// Component registration
export const basicComponentLibrary: ComponentLibrary = {
  name: 'basic',
  elements: [
    {
      tagName: 'ui-button',
      component: UIButton,
      propMapping: { label: 'label' },
      eventMapping: { press: 'onPress' }
    }
  ]
};
```

### Remote Element Configuration

```typescript
interface RemoteElementConfiguration {
  tagName: string;
  remoteAttributes?: string[];  // Attributes to sync
  remoteEvents?: string[];      // Events to forward
}

// Usage
<RemoteDOMResourceRenderer
  resource={resource}
  remoteElements={[
    { tagName: 'ui-button', remoteEvents: ['press'] },
    { tagName: 'ui-input', remoteAttributes: ['value'], remoteEvents: ['change'] }
  ]}
/>
```

## Security Implementation

### Iframe Sandboxing

```typescript
// Minimal permissions for HTML content
sandbox="allow-scripts"

// Additional permission for external URLs
sandbox="allow-scripts allow-same-origin"

// Remote DOM uses custom bundled runtime
// No external scripts can be loaded
```

### Message Validation

```typescript
useEffect(() => {
  function handleMessage(event: MessageEvent) {
    // Only process messages from our iframe
    if (iframeRef.current && event.source === iframeRef.current.contentWindow) {
      const uiActionResult = event.data as UIActionResult;
      if (!isValidUIAction(uiActionResult)) return;
      
      onUIAction?.(uiActionResult)?.catch((err) => {
        console.error('Error handling UI action:', err);
      });
    }
  }
  window.addEventListener('message', handleMessage);
  return () => window.removeEventListener('message', handleMessage);
}, [onUIAction]);
```

## Resource Processing

### Content Extraction

```typescript
function processResource(resource: Partial<Resource>): ProcessResourceResult {
  // Handle text content
  if (resource.text) {
    return { htmlString: resource.text, iframeRenderMode: 'srcDoc' };
  }
  
  // Handle blob content (base64 encoded)
  if (resource.blob) {
    const decoded = new TextDecoder().decode(
      Uint8Array.from(atob(resource.blob), c => c.charCodeAt(0))
    );
    return { htmlString: decoded, iframeRenderMode: 'srcDoc' };
  }
  
  // Handle URL lists
  if (resource.mimeType === 'text/uri-list') {
    const urls = parseUriList(resource.text);
    return { iframeSrc: urls[0], iframeRenderMode: 'src' };
  }
}
```

## Build System

### Monorepo Structure

```yaml
# pnpm-workspace.yaml
packages:
  - 'packages/*'
  - 'examples/*'
```

### Package Dependencies

- **Server**: Minimal dependencies, just MCP SDK
- **Client**: React, Remote DOM libraries, threading utilities
- **Examples**: Full application dependencies

## Performance Considerations

### Resource Delivery

```typescript
// For small content (<100KB)
delivery: 'text'

// For larger content
delivery: 'blob'  // Base64 encoded

// Encoding helper
function robustUtf8ToBase64(str: string): string {
  if (typeof Buffer !== 'undefined') {
    return Buffer.from(str, 'utf-8').toString('base64');
  }
  // Fallback for browser environments
  const encoder = new TextEncoder();
  const uint8Array = encoder.encode(str);
  let binaryString = '';
  uint8Array.forEach((byte) => {
    binaryString += String.fromCharCode(byte);
  });
  return btoa(binaryString);
}
```

## Testing Approach

### Server Tests
- Resource creation validation
- MIME type handling
- Encoding/decoding accuracy

### Client Tests
- Component rendering
- Message handling
- Security boundary enforcement
- Resource type detection

## Common Integration Patterns

### 1. Form Submission
```typescript
// In UI resource
<form onsubmit="event.preventDefault(); parent.postMessage({type: 'tool', payload: {toolName: 'submitForm', params: new FormData(this)}}, '*')">
```

### 2. Real-time Updates
```typescript
// Server sends updated resource
// Client re-renders with new content
// State managed on server side
```

### 3. Multi-step Workflows
```typescript
// Each step returns new UI resource
// Navigation handled via tool calls
// Progress tracked server-side
```

This technical deep-dive should give you a solid understanding of how MCP-UI works internally and how to extend it as a contributor.