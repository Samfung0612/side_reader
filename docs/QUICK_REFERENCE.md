# Component Quick Reference Card

## 🎯 At a Glance

| Component | Responsibility | Key State | User Interaction |
|-----------|-----------------|-----------|-------------------|
| **App.tsx** | Security & layout | `isUnlocked`, `showUnlockModal` | None (auto-runs) |
| **Header.tsx** | Controls & refresh | `pageContext`, `isRefreshing` | Click refresh, settings, new chat |
| **ChatArea.tsx** | Display messages | `messages`, `isGenerating` | Scroll, copy code |
| **InputArea.tsx** | Send messages | `input`, `stagedAttachments` | Type, upload, send, toggle context |
| **Settings.tsx** | API config | `apiProvider`, `providerConfigs` | Select provider, enter key, pick model |
| **HistoryDrawer.tsx** | Switch sessions | `sessions`, `currentSessionId` | Select/delete session |
| **UnlockModal.tsx** | Unlock keys | `password` input | Enter master password |
| **AgentManager.tsx** | Manage AI agents | `agents`, `currentAgent` | Create, edit, delete agents |
| **AgentDrawer.tsx** | Select agent | `agents` | Click to switch agent |

---

## 📊 Data Structure Reference

### Message
```typescript
{
  id: string                      // UUID
  role: 'user' | 'assistant'
  content: string                 // Rendered as Markdown
  reasoningContent?: string       // Extended thinking
  timestamp: number               // Unix time
  attachedContext?: {
    title: string                 // Page title
    url: string                   // Page URL
    snapshotId: string            // IndexedDB key
  }
  attachments?: [{                // User files
    id: string
    name: string
    mimeType: string
  }]
  stopped?: boolean               // User aborted
}
```

### Session
```typescript
{
  id: string                      // UUID
  title: string                   // Auto-generated
  updatedAt: number               // Last modified
  messages: Message[]
  agentId?: string                // Linked AI agent
}
```

### Agent
```typescript
{
  id: string
  name: string                    // e.g., "Code Expert"
  prompt: string                  // System prompt
  icon: string                    // Icon key
  inputPlaceholder: string        // Dynamic hint
  isBuiltIn: boolean
}
```

---

## 🔄 Main Flows

### 1. Extracting Page Context
```
Header.tsx
  ↓ User clicks refresh
  ↓ extractContext()
  ↓ Get active tab URL
  ↓ Execute content script
  ↓ Parse HTML with DOMParser
  ↓ Mozilla Readability extracts text
  ↓ setPageContext() → store
  ↓ Save snapshot → IndexedDB
```

### 2. Sending a Message
```
InputArea.tsx
  ↓ User types + clicks send
  ↓ addMessage() → store
  ↓ generateResponse()
    ├─ Build AI payload (history + context)
    ├─ Call Gemini or OpenAI API
    ├─ Stream response
    └─ addMessageChunk()
  ↓ ChatArea renders streaming text
  ↓ Save to chrome.storage
```

### 3. File Upload
```
InputArea.tsx
  ↓ User selects file
  ↓ Convert to Base64
  ↓ Create preview
  ↓ Stage in stagedAttachments
  ↓ Show preview thumbnail
  ↓ On send: saveAttachmentBlob() → IndexedDB
  ↓ Attach metadata to message
```

### 4. Unlocking Encryption
```
App.tsx
  ↓ Check hasMasterPassword()
  ↓ If yes: show UnlockModal
  ↓ User enters password
  ↓ SecureStorage.unlock(password)
    ├─ Verify against stored hash
    ├─ Derive key from password
    ├─ Decrypt API key cache
    └─ Store in memory
  ↓ setIsUnlocked(true)
  ↓ Close modal
```

---

## 🎛️ Zustand Store API

```typescript
// Current session
currentSessionId: string
getCurrentSession(): Session

// Messages
messages: Message[]
addMessage(message: Message): void
addMessageChunk(chunk: string): void

// Settings
apiProvider: 'gemini' | 'openai'
providerConfigs: { gemini: {...}, openai: {...} }

// Page context
pageContext: ContextMeta | null
setPageContext(context: ContextMeta | null): void
useContext: boolean                           // Toggle switch
setUseContext(use: boolean): void

// Agents
agents: Agent[]
currentAgentId: string
getCurrentAgent(): Agent

// Generation
isGenerating: boolean
abortController: AbortController | null
generateResponse(): Promise<void>

// UI
isSettingsOpen: boolean
isDrawerOpen: boolean
isAgentDrawerOpen: boolean
setIsSettingsOpen(open: boolean): void
setIsDrawerOpen(open: boolean): void
setIsAgentDrawerOpen(open: boolean): void
```

---

## 🔐 Security Checklist

- ✅ API keys encrypted at rest (AES-GCM)
- ✅ Master password optional (PBKDF2 derivation)
- ✅ Session-based unlock (expires on browser close)
- ✅ No external logging of keys
- ✅ Content script restricted URLs (chrome://, edge://, about:)
- ✅ Markdown sanitized (DOMPurify)
- ✅ All storage local-only (no cloud sync)

---

## 🛠️ Common Development Tasks

### Add a New Component
```typescript
// 1. Create file
src/components/MyComponent.tsx

// 2. Use store if needed
import { useStore } from '../store'

// 3. Add to App.tsx
import { MyComponent } from './components/MyComponent'
<MyComponent />

// 4. Style with Tailwind
<div className="flex gap-2 dark:bg-gray-800">
```

### Access Zustand State
```typescript
const { 
  currentSessionId, 
  messages, 
  apiProvider 
} = useStore()
```

### Add New Store State
```typescript
// In src/store/index.ts
const useStore = create<State>()(
  persist(
    (set) => ({
      // Add new field
      myField: 'value',
      setMyField: (val) => set({ myField: val })
    }),
    {
      name: 'luminasider-store',
      storage: createJSONStorage(() => chromeStorage)
    }
  )
)
```

### Handle Streaming Response
```typescript
// In InputArea or custom handler
const response = await fetch(apiUrl, {
  method: 'POST',
  headers: { 'accept': 'text/event-stream' }
})

const reader = response.body?.getReader()
while (true) {
  const { done, value } = await reader?.read() || {}
  if (done) break
  const chunk = new TextDecoder().decode(value)
  addMessageChunk(chunk)
}
```

---

## 📚 Where to Find Things

| Question | Location |
|----------|----------|
| How does context extraction work? | COMPONENT_USAGE.md → Header.tsx |
| Where is sensitive data stored? | PROJECT_STRUCTURE.md → Storage Details |
| What's the tech stack? | PROJECT_DESCRIPTION.md → Technology Stack |
| How to run dev server? | .github/copilot-instructions.md |
| What's the file structure? | PROJECT_STRUCTURE.md → Directory Layout |
| How do I add a provider? | COMPONENT_USAGE.md → Settings.tsx |
| What are the encryption details? | COMPONENT_USAGE.md → secureStorage.ts |

---

## 🚨 Common Pitfalls

| Mistake | Problem | Solution |
|---------|---------|----------|
| Direct localStorage access | Breaks persistence | Use Zustand store only |
| Creating elements in DOM | React doesn't track it | Use React components & JSX |
| Blocking content script | Extension freezes | Use async/await |
| Hardcoded API URLs | Can't proxy | Use configurable baseUrl |
| Logging API keys | Security risk | Never log credentials |
| Exceeding IndexedDB quota | Upload fails silently | Implement cleanup |

---

**For detailed documentation, see:**
- `docs/README.md` - Navigation hub
- `docs/PROJECT_DESCRIPTION.md` - Architecture overview
- `docs/COMPONENT_USAGE.md` - Detailed component guide
- `docs/PROJECT_STRUCTURE.md` - File layout & organization
