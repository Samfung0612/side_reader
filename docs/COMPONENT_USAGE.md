# Component Usage Guide

## Overview
LuminaSider is built with modular React components following a feature-based organization. Each component handles a specific UI area or functionality.

---

## Core Components

### 1. **App.tsx** - Main Application Container
**Location**: `src/App.tsx`

**Purpose**: Root component that initializes the entire extension, handles security, and renders the layout.

**Responsibilities**:
- Check master password security status on mount
- Show unlock modal if password-protected but not unlocked
- Render main layout structure (Header → ChatArea → InputArea)
- Manage auth state

**Key State**:
```typescript
isUnlocked: boolean      // Whether API keys are accessible
hasMasterPassword: bool  // Whether user set a password
showUnlockModal: bool    // Show/hide password entry modal
```

**Usage Pattern**:
```typescript
// App checks security, then renders UI
if (hasMasterPassword && !isUnlocked) {
  <UnlockModal onUnlock={handleUnlock} />
}
```

**Dependencies**: Zustand store, React hooks

---

### 2. **Header.tsx** - Top Control Bar
**Location**: `src/components/Header.tsx`

**Purpose**: Displays session info, controls, and context refresh button. Extracts current webpage content.

**Responsibilities**:
- Display current session title
- Show current AI agent icon & name
- Extract page context when user clicks refresh
- Toggle settings panel
- Trigger new session creation
- Show history drawer

**Key Functions**:
```typescript
extractContext()       // Main function - extracts page content using Readability
handleRefresh()        // User clicked refresh button
handleNewSession()     // Create blank conversation
```

**How Context Extraction Works**:
1. Get active tab URL (skip `chrome://` URLs)
2. Execute content script to get page HTML
3. Parse HTML with DOMParser
4. Use Mozilla Readability to extract clean article text
5. Store in Zustand (title, content, url)
6. Save snapshot ID to IndexedDB for large content

**UI Elements**:
- Session title display
- Agent icon selector
- Refresh button (with loading spinner)
- Settings icon (⚙️)
- History icon (📖)
- New session button (➕)

**Zustand Access**:
```typescript
const { pageContext, setPageContext, isSettingsOpen, setIsSettingsOpen } = useStore();
```

---

### 3. **ChatArea.tsx** - Message Display & Streaming
**Location**: `src/components/ChatArea.tsx`

**Purpose**: Renders conversation history with real-time streaming response support and attachment previews.

**Responsibilities**:
- Display message history (user & assistant messages)
- Render streaming responses with typing effect
- Show attached images/files inline
- Support copy-to-clipboard for code blocks
- Syntax highlighting for code

**Key Components**:
- `AttachmentView`: Renders image/file attachments with lazy loading
- `CodeBlock`: Custom code block renderer with copy button & syntax highlighting
- `ReactMarkdown`: Renders Markdown with custom code block handler

**Message Structure**:
```typescript
interface Message {
  id: string
  role: 'user' | 'assistant'
  content: string           // Main message text
  reasoningContent?: string // Extended thinking from AI
  timestamp: number
  attachedContext?: {       // Injected webpage context
    title: string
    url: string
    snapshotId: string
  }
  attachments?: Array<{     // User-uploaded files
    id: string
    name: string
    mimeType: string
  }>
  stopped?: boolean         // User aborted generation
}
```

**Streaming Behavior**:
- Uses `react-markdown` to progressively render text
- Supports partial HTML rendering as chunks arrive
- Auto-scrolls to latest message
- Shows typing indicator while generating

**Code Block Features**:
- Language detection (JavaScript, Python, etc.)
- Syntax highlighting via highlight.js
- Copy button with confirmation (✓ appears briefly)
- Line wrapping enabled

---

### 4. **InputArea.tsx** - Message Input & File Upload
**Location**: `src/components/InputArea.tsx`

**Purpose**: Text input, file attachment handling, and message submission with AI generation.

**Responsibilities**:
- Text input with auto-expanding height
- File upload (images, documents)
- Toggle context attachment ("Include page content" switch)
- Manage staged attachments
- Submit messages to AI provider
- Handle streaming response cancellation

**Key Features**:
- **Auto-expanding textarea**: Grows to max 120px, then scrolls
- **Staged attachments**: Files queued before sending (with preview)
- **Context toggle**: Checkbox to include current page content
- **Agent placeholder**: Dynamic placeholder text per AI agent
- **Secure API key loading**: Checks master password unlock status

**File Upload Flow**:
1. User clicks file icon or pastes file
2. Convert file to Base64
3. Create preview (for images)
4. Stage in `stagedAttachments` array
5. On send: save to IndexedDB, get back attachment ID
6. Attach metadata to message

**Keyboard Controls**:
```
Enter              → Send message
Shift + Enter      → New line
Ctrl/Cmd + Enter   → Alternative send (configurable)
```

**Message Generation Flow**:
1. Collect user input, attachments, context
2. Build API payload with message history
3. Check if API key is unlocked (if password-protected)
4. Send to selected provider (Gemini or OpenAI-compatible)
5. Stream response back → ChatArea renders in real-time
6. User can click stop button to abort

---

### 5. **Settings.tsx** - API Configuration Panel
**Location**: `src/components/Settings.tsx`

**Purpose**: Configure AI provider, API keys, models, and advanced options.

**Responsibilities**:
- Switch between providers (Gemini ↔ OpenAI)
- Input/edit API keys securely
- Fetch available models from provider
- Select model
- Set custom base URL (for proxies)
- Configure temperature & other parameters
- Optional: Set master password for key encryption

**Provider-Specific Configs**:

**Google Gemini**:
- API key required
- Models fetched from Google API
- No custom URL option

**OpenAI-Compatible**:
- API key required
- Base URL customizable (default: https://api.openai.com/v1)
- Supports proxies (e.g., https://api.deepseek.com/v1)
- Models: user manually enters or selects from list

**Security**:
- API key stored in `chrome.storage.local` (encrypted if password set)
- Master password option: derives key using PBKDF2 + AES-GCM
- Session-based unlock: caches decrypted key in memory

**UI Flow**:
1. User selects provider
2. Inputs API key (field marked sensitive)
3. (Optional) Clicks "Get Models" to fetch list
4. Selects model
5. Tests connection (optional)
6. Saves settings → auto-persisted to store

---

### 6. **HistoryDrawer.tsx** - Session History
**Location**: `src/components/HistoryDrawer.tsx`

**Purpose**: List previous conversations and switch between sessions.

**Responsibilities**:
- Display list of all sessions (title, last updated)
- Switch to a session (load messages)
- Delete a session
- Create new session
- Search/filter sessions (optional)

**Session Structure**:
```typescript
interface Session {
  id: string
  title: string          // Auto-generated or user-set
  updatedAt: number      // Timestamp
  messages: Message[]
  agentId?: string       // Linked AI agent
}
```

**UI Elements**:
- Searchable session list
- Last updated time
- Delete button (with confirm)
- Session selection (highlight current)

---

### 7. **UnlockModal.tsx** - Master Password Entry
**Location**: `src/components/UnlockModal.tsx`

**Purpose**: Prompt user to enter master password to unlock API keys.

**Responsibilities**:
- Display password input field
- Validate against stored password hash
- Unlock decrypted API key cache on success
- Show error message on wrong password

**Security Flow**:
1. User enters master password
2. Compare against hash stored in `chrome.storage.local`
3. If match: call `SecureStorage.unlock(password)` → decrypts cache
4. Cache valid for entire browser session
5. On modal close: keys accessible to Settings & InputArea

---

### 8. **AgentManager.tsx** - AI Agent Configuration
**Location**: `src/components/AgentManager.tsx`

**Purpose**: Create, edit, and manage custom AI agents with custom prompts.

**Responsibilities**:
- List all agents (built-in + custom)
- Edit agent prompt, name, icon
- Test agent (run quick query)
- Delete custom agent
- Set agent as active

**Agent Structure**:
```typescript
interface Agent {
  id: string
  name: string              // e.g., "Code Expert"
  prompt: string            // System prompt injected into every message
  icon: string              // Icon key (BookOpen, Code, Bot, Languages)
  inputPlaceholder: string  // Dynamic placeholder text
  isBuiltIn: boolean
}
```

**Custom Agent Example**:
- Name: "Code Reviewer"
- Prompt: "You are an expert code reviewer. Always review for security, performance, and readability."
- Icon: "Code"

---

### 9. **AgentDrawer.tsx** - Agent Selection Panel
**Location**: `src/components/AgentDrawer.tsx`

**Purpose**: Quick-switch between AI agents.

**Responsibilities**:
- Display list of available agents
- Show agent icon, name, and description
- Switch active agent
- Open AgentManager for editing

**UI Elements**:
- Agent cards with icons
- Selected agent highlight
- "Manage Agents" link

---

## Utility Functions & Helpers

### **secureStorage.ts** - Encryption & Key Management
**Location**: `src/utils/secureStorage.ts`

**Purpose**: Encrypt/decrypt API keys using master password with AES-GCM.

**Key Functions**:
```typescript
// Setup
setMasterPassword(password: string)    // Create master password
hasMasterPassword(): Promise<bool>     // Check if password exists

// Usage
unlock(password: string): Promise<bool>  // Verify & unlock
lock()                                   // Lock (clear cache)
checkUnlocked(): Promise<bool>           // Is currently unlocked?

// Storage
saveEncrypted(key: string, plaintext: string)   // Encrypt & save
getDecrypted(key: string): Promise<string>      // Load & decrypt
```

**Encryption Details**:
- Algorithm: AES-GCM (128-bit encryption)
- Key Derivation: PBKDF2 (100,000 iterations, SHA-256)
- Salt: 16 bytes random
- IV (Nonce): 12 bytes random
- Session Cache: In-memory map (cleared on browser close)

---

### **Zustand Store** - Global State
**Location**: `src/store/index.ts`

**Purpose**: Centralized state management with persistence.

**Key State Sections**:
```typescript
// Session & Messages
currentSessionId: string
sessions: Session[]
messages: Message[]

// API Configuration
apiProvider: 'gemini' | 'openai'
providerConfigs: {
  gemini: { apiKey: string, model: string }
  openai: { apiKey: string, model: string, baseUrl: string }
}

// UI State
isSettingsOpen: boolean
isDrawerOpen: boolean      // History drawer
isAgentDrawerOpen: boolean
pageContext: ContextMeta | null

// Agents
agents: Agent[]
currentAgentId: string

// Generation
isGenerating: boolean
abortController: AbortController | null
```

**Persistence**:
- Middleware auto-saves to `chrome.storage.local`
- Large data (snapshots, attachments) → IndexedDB
- Session cache expires on browser restart

---

## Data Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    App.tsx                              │
│            (Check Security, Render Layout)              │
└────────────────┬────────────────────────────────────────┘
                 │
        ┌────────┴──────────┬──────────────┬──────────────┐
        │                   │              │              │
        ▼                   ▼              ▼              ▼
    Header.tsx         ChatArea.tsx    InputArea.tsx  Settings.tsx
 (Extract Context)  (Show Messages)  (Send Messages)  (API Config)
        │                   │              │              │
        └────────┬──────────┴──────────────┴──────────────┘
                 │
        ┌────────▼──────────┐
        │  Zustand Store    │
        │  (State & Logic)  │
        └────────┬──────────┘
                 │
        ┌────────┴──────────┐
        │                   │
        ▼                   ▼
  chrome.storage      IndexedDB
  (Config, Keys)  (Large Content)
        │                   │
        │    ┌──────────────┘
        │    │
        └────▼──────────────┐
                            │
                    ▼       ▼
                Gemini / OpenAI API
                  (Generate Response)
```

---

## Best Practices

### When Adding a New Component
1. Keep components focused on one responsibility
2. Use Zustand for shared state (avoid prop drilling)
3. Extract custom logic into `src/utils/` helpers
4. Use TypeScript for all props & state
5. Add JSDoc comments for public functions
6. Use Tailwind classes; avoid inline styles

### Performance Tips
- Lazy-load large attachments (images in ChatArea)
- Use `useCallback` to prevent unnecessary re-renders
- Memoize expensive computations (context extraction)
- Keep message history in IndexedDB, paginate in UI

### Security Checklist
- Never log API keys
- Validate all user input before sending to AI
- Sanitize Markdown output (DOMPurify already integrated)
- Check tab URLs (exclude restricted URLs)
- Use async/await for storage operations

---

For setup and development commands, see [../.github/copilot-instructions.md](../.github/copilot-instructions.md)
