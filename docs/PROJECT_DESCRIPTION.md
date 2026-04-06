# LuminaSider - Project Description

## 📋 Overview

**LuminaSider** is a Chrome extension (Manifest V3) that transforms web browsing into an intelligent, context-aware experience. It injects an AI-powered sidebar into every webpage, allowing users to:

- **Read & Understand**: Extract and analyze web page content using Mozilla Readability
- **Ask Questions**: Get precise answers about the current webpage with full context awareness
- **Translate & Summarize**: Use AI to translate text or create concise summaries
- **Work with Multiple AI Providers**: Support for Google Gemini, OpenAI, DeepSeek, Claude, and any OpenAI-compatible API
- **Upload Files**: Add images and documents to queries for multimodal AI analysis
- **Maintain Privacy**: All API keys stored locally in the browser; no external server required

## 🏗️ Architecture & Data Flow

### Core Components

```
Extension Structure:
├── Background Worker (src/background/index.ts)
│   └── Manages tab lifecycle, side panel behavior, extension initialization
├── Content Script (src/content/index.ts)
│   └── Injects into all pages; extracts page content
└── Side Panel UI (React App)
    ├── Header: Controls, agent selection, page context refresh
    ├── ChatArea: Displays conversation history with streaming responses
    ├── InputArea: Message input with file upload & context toggle
    ├── Settings: API configuration & model selection
    └── Utilities: Secure storage, helpers
```

### Data Storage Strategy

| Data Type | Storage | Purpose | Reason |
|-----------|---------|---------|--------|
| **API Credentials** | `chrome.storage.local` | Encrypted API keys | Fast, small footprint |
| **Page Snapshots** | IndexedDB (`idb-keyval`) | Large webpage content | Prevents main thread blocking |
| **Chat History** | `chrome.storage.local` | Session messages | Persistence across sessions |
| **Attachments** | IndexedDB | Images, files (base64) | Handles large binary data |

This decoupling ensures the extension remains responsive even when processing long articles or multiple image attachments.

## 🔄 Message Flow

```
1. User clicks extension icon
   ↓
2. Background worker opens side panel
   ↓
3. User toggles "Attach Current Page" in InputArea
   ↓
4. Header.extractContext() executes content script
   ↓
5. Content script reads page HTML and sends to Header
   ↓
6. Header uses Mozilla Readability to extract clean content
   ↓
7. Context stored in Zustand store → IndexedDB (for snapshots)
   ↓
8. Message + Context sent to AI provider (Gemini, OpenAI, etc.)
   ↓
9. Streaming response received & rendered with Markdown + Syntax Highlighting
```

## 🎨 UI/UX Principles

- **Minimal & Clean**: Tailwind CSS + lucide-react icons
- **Dark Mode Support**: `dark:` prefix for responsive theming
- **Responsive**: Adapts to sidebar constraints (typically 350-400px width)
- **Streaming UI**: Real-time message rendering as AI responds
- **Keyboard-First**: Shift+Enter for newline, Enter to send

## 🔒 Security

- **Master Password**: Optional password protection for API keys
- **AES-GCM Encryption**: API keys encrypted at rest using PBKDF2 key derivation
- **Session-Based Unlock**: Keys decrypt on unlock, cache expires with browser session
- **No External Logging**: All processing happens locally or through user-selected API provider
- **URL Filtering**: Excludes `chrome://`, `edge://`, and `about:` pages from content extraction

## 🛠️ Technology Stack

- **Frontend**: React 18 + TypeScript
- **Build**: Vite + @crxjs/vite-plugin (handles Manifest V3 bundling)
- **State**: Zustand with persistence middleware
- **Styling**: Tailwind CSS + tailwindcss-animate
- **Content Extraction**: @mozilla/readability (robust article extraction)
- **Rendering**: react-markdown + highlight.js (code syntax highlighting)
- **Storage**: chrome.storage.local + IndexedDB (idb-keyval)
- **Encryption**: Web Crypto API (AES-GCM)
- **Icons**: lucide-react

## 📦 Deployment

```bash
# Development
npm run dev          # Runs Vite dev server with HMR
                     # Load dist/ folder as unpacked extension

# Production
npm run build        # TypeScript check + Vite build
npm run pack         # Creates luminasider-extension.zip
```

## 🔌 Supported AI Providers

1. **Google Gemini**
   - Models: gemini-1.5-flash, gemini-1.5-pro, gemini-2.0-flash, etc.
   - Multimodal support (text + images)

2. **OpenAI-Compatible APIs**
   - OpenAI (GPT-4, GPT-4o, etc.)
   - DeepSeek
   - Claude (via OpenAI-compatible interface)
   - Any provider with OpenAI API format support
   - Custom base URL support for proxies/self-hosted models

## 🚀 Key Features

- ✅ **Web Page Context**: Automatically extract current page content
- ✅ **Streaming Responses**: Real-time AI output with typing effect
- ✅ **Multimodal Input**: Paste/upload images directly in chat
- ✅ **Session History**: Save and restore conversations
- ✅ **Custom Agents**: Create AI agents with custom prompts & instructions
- ✅ **Markdown Rendering**: Full Markdown support with code highlighting
- ✅ **Dark Mode**: Light and dark theme support
- ✅ **Privacy-First**: All data stored locally; no cloud sync

---

For detailed component documentation, see [COMPONENT_USAGE.md](./COMPONENT_USAGE.md)

For development guidelines, see [../.github/copilot-instructions.md](../.github/copilot-instructions.md)
