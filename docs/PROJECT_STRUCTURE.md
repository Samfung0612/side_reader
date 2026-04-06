# Project Structure

## Directory Layout

```
side_reader/
├── .github/
│   ├── workflows/          # GitHub Actions CI/CD (if any)
│   └── copilot-instructions.md  # Instructions for AI assistants
│
├── docs/                   # 📚 Documentation
│   ├── PROJECT_DESCRIPTION.md   # Overview & architecture
│   ├── COMPONENT_USAGE.md       # Detailed component guide
│   └── PROJECT_STRUCTURE.md     # This file
│
├── public/
│   └── icons/              # Extension icons (16x16, 48x48, 128x128)
│       ├── icon16.png
│       ├── icon48.png
│       └── icon128.png
│
├── src/
│   ├── App.tsx             # Root component (security, layout)
│   ├── main.tsx            # React DOM entry point
│   ├── index.css           # Global styles
│   ├── vite-env.d.ts       # Vite type definitions
│   │
│   ├── background/
│   │   └── index.ts        # Service worker (tab lifecycle, messaging)
│   │
│   ├── content/
│   │   └── index.ts        # Content script (inject into pages)
│   │
│   ├── components/
│   │   ├── Header.tsx      # Top bar (controls, context extraction)
│   │   ├── ChatArea.tsx    # Message display & streaming
│   │   ├── InputArea.tsx   # Text input & file upload
│   │   ├── Settings.tsx    # API configuration panel
│   │   ├── HistoryDrawer.tsx  # Session history switcher
│   │   ├── UnlockModal.tsx    # Master password entry
│   │   ├── AgentManager.tsx   # AI agent editor
│   │   └── AgentDrawer.tsx    # Agent selector
│   │
│   ├── store/
│   │   └── index.ts        # Zustand state management + persistence
│   │
│   └── utils/
│       └── secureStorage.ts # AES-GCM encryption for API keys
│
├── index.html              # Main HTML entry (side panel)
├── manifest.json           # Chrome extension manifest (Manifest V3)
├── package.json            # Dependencies & npm scripts
├── tsconfig.json           # TypeScript configuration (strict mode)
├── vite.config.ts          # Vite + Crxjs plugin config
├── tailwind.config.js      # Tailwind CSS configuration
├── postcss.config.js       # PostCSS config (for Tailwind)
├── .gitignore              # Git ignore rules
├── README.md               # User-facing README
├── CHANGELOG.md            # Version history
└── LICENSE                 # MIT License
```

---

## File Descriptions

### Configuration Files

| File | Purpose |
|------|---------|
| `manifest.json` | Extension permissions, permissions, background worker, content scripts, icons |
| `tsconfig.json` | TypeScript strict mode, JSX, module resolution |
| `vite.config.ts` | Vite build config + @crxjs/vite-plugin for Manifest V3 bundling |
| `tailwind.config.js` | Tailwind theme, animations, dark mode settings |
| `postcss.config.js` | PostCSS plugins (required for Tailwind) |
| `package.json` | Dependencies, npm scripts (dev, build, pack, preview) |

### Entry Points

| File | Purpose | Output |
|------|---------|--------|
| `src/main.tsx` | React DOM render (side panel) | Injects React into `index.html` |
| `src/background/index.ts` | Service worker | Runs in extension background (no UI) |
| `src/content/index.ts` | Content script | Injects into every webpage |

### UI Components (in `src/components/`)

```
App.tsx                Root layout, security check
├── Header.tsx         Controls, context extraction, refresh
├── ChatArea.tsx       Message display, streaming, code highlighting
├── InputArea.tsx      Message input, file upload, context toggle
├── HistoryDrawer.tsx  Session switcher
├── UnlockModal.tsx    Master password entry
├── AgentDrawer.tsx    Agent switcher
└── Settings.tsx       API provider config
    └── AgentManager.tsx (modal within Settings)
```

### State & Storage

| File | Responsibility |
|------|-----------------|
| `src/store/index.ts` | Zustand store, persistence middleware, API interfaces |
| `src/utils/secureStorage.ts` | AES-GCM encryption, master password derivation |

---

## Data Flow Between Layers

```
┌─────────────────────────────────────┐
│         Chrome Extension            │
│  (Manifest V3 Permissions)          │
├─────────────────────────────────────┤
│ Background Worker (src/background/) │
│ • Tab lifecycle (activate, update)  │
│ • Open side panel on click          │
│ • Forward messages between contexts │
├─────────────────────────────────────┤
│ Content Script (src/content/)       │
│ • Runs on every webpage             │
│ • Extracts page HTML on demand      │
│ • Communicates with side panel      │
├─────────────────────────────────────┤
│ Side Panel (React App)              │
│ • User interface (Header, ChatArea) │
│ • Zustand state management          │
│ • Chrome storage integration        │
├─────────────────────────────────────┤
│ Storage Layer                       │
│ • chrome.storage.local (config)     │
│ • IndexedDB (large content)         │
├─────────────────────────────────────┤
│ External APIs                       │
│ • Google Gemini API                 │
│ • OpenAI-compatible endpoints       │
└─────────────────────────────────────┘
```

---

## Build Output

```
dist/                          # Final extension bundle
├── index.html                 # Main side panel HTML (processed by Crxjs)
├── manifest.json              # Manifest V3 (processed)
├── assets/
│   ├── [main chunk].js        # React app bundle
│   ├── [background].js        # Service worker
│   ├── [content].js           # Content script
│   └── [vendor chunks].js     # Dependencies (react, zustand, etc.)
└── public/
    └── icons/                 # Icon assets
```

---

## How to Navigate the Codebase

### 1. **Understanding the Flow of a Message**
- Start: `src/App.tsx` → check auth
- User types: `src/components/InputArea.tsx` → `useState` for input
- On send: → `useStore().addMessage()` → Zustand updates store
- Store triggers: → `generateResponse()` (calls AI provider)
- Response streams: → `src/components/ChatArea.tsx` displays chunks
- Saved: → `chrome.storage.local` + IndexedDB (via persistence middleware)

### 2. **Adding a New Component**
1. Create `src/components/NewFeature.tsx`
2. Import Zustand hooks if needed: `import { useStore } from '../store'`
3. Use Tailwind for styling
4. Export component
5. Import & render in `App.tsx`

### 3. **Adding API Integration**
1. Extend `providerConfigs` in `src/store/index.ts`
2. Add API call logic to `generateResponse()` function
3. Handle streaming response (SSE or WebSocket)
4. Emit chunks to `addMessageChunk()` for real-time display

### 4. **Modifying Styling**
- Tailwind classes in JSX components
- Dark mode: use `dark:` prefix
- Color vars in `tailwind.config.js` if custom palette needed

### 5. **Debugging Storage Issues**
- Chrome DevTools → Storage tab → `chrome.storage.local`
- Application tab → IndexedDB → `idb-keyval` store
- Check network tab for API calls to Gemini/OpenAI

---

## Key Dependencies

### UI & Rendering
- **react** (18.2.0) - Component framework
- **react-dom** (18.2.0) - DOM rendering
- **react-markdown** (9.0.1) - Markdown rendering
- **highlight.js** (11.9.0) - Code syntax highlighting

### State & Storage
- **zustand** (4.5.2) - Lightweight state management
- **idb-keyval** (6.2.2) - IndexedDB wrapper
- **@types/chrome** (0.0.268) - Chrome API types

### AI & Content
- **@google/generative-ai** (0.24.1) - Google Gemini API
- **@mozilla/readability** (0.5.0) - Article content extraction
- **dompurify** (3.3.3) - HTML sanitization

### Styling & Icons
- **tailwindcss** (3.4.3) - Utility-first CSS
- **tailwindcss-animate** (1.0.7) - Animation utilities
- **lucide-react** (0.378.0) - Icon library
- **clsx** (2.1.1) - Conditional class composition
- **tailwind-merge** (3.5.0) - Merge Tailwind classes

### Build Tools
- **vite** (5.2.0) - Fast bundler
- **@crxjs/vite-plugin** (2.0.0-beta.23) - Chrome extension support
- **@vitejs/plugin-react** (4.2.1) - React Fast Refresh
- **typescript** (5.2.2) - Type checking

---

## npm Scripts

```bash
npm run dev      # Start Vite dev server (watch mode)
npm run build    # TypeScript check + Vite production build
npm run preview  # Preview production build locally
npm run pack     # Build + zip dist/ for release
```

---

## Storage Details

### `chrome.storage.local`
**Size Limit**: ~10MB per extension  
**Use Cases**: API keys (encrypted), settings, session metadata

**Stored by store persistence**:
```typescript
{
  state: {
    currentSessionId: "...",
    sessions: [...],
    apiProvider: "gemini",
    providerConfigs: {...},
    agents: [...],
    ...
  }
}
```

### IndexedDB (`idb-keyval`)
**Size Limit**: Varies by browser (typically 50MB+)  
**Use Cases**: Large page snapshots, attachment files (base64)

**Keys**:
```
"snapshot_{id}"        // Page content snapshots
"attachment_{id}"      // Image/file attachments (base64)
```

---

## Environment Variables

Currently **no `.env` file required** for basic development.

For production, optional:
```env
# Not used by extension itself, but useful for build scripts
VITE_API_URL=https://api.example.com
VITE_DEBUG=false
```

---

## Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| Extension not loading | Missing manifest.json | Run `npm run build` first |
| API key errors | Wrong provider selected | Check Settings panel |
| Page context not extracting | Restricted URL (chrome://, etc.) | Only works on regular pages |
| Large file uploads fail | IndexedDB quota exceeded | Clear old sessions/attachments |
| Streaming stops abruptly | Network interruption | Check browser network tab |

---

For more details on specific components, see [COMPONENT_USAGE.md](./COMPONENT_USAGE.md)

For architecture & data flow, see [PROJECT_DESCRIPTION.md](./PROJECT_DESCRIPTION.md)

For development guidelines, see [../.github/copilot-instructions.md](../.github/copilot-instructions.md)
