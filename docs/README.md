# LuminaSider Documentation Index

Welcome! This folder contains comprehensive documentation for the LuminaSider Chrome extension project.

## 📚 Documentation Files

### **[PROJECT_DESCRIPTION.md](./PROJECT_DESCRIPTION.md)** - Start Here!
High-level overview of the project:
- What LuminaSider does
- Core architecture & data flow
- Technology stack
- Supported AI providers
- Key security features

**Best for**: Understanding the project purpose, overall design, and capabilities.

---

### **[COMPONENT_USAGE.md](./COMPONENT_USAGE.md)** - Deep Dive into Code
Detailed guide to every React component and utility:
- Component responsibilities & API
- How context extraction works
- Message flow & streaming UI
- File upload handling
- Settings & encryption
- Data structures (Message, Session, Agent, etc.)
- Best practices for development

**Best for**: Working on features, adding new components, understanding data flow.

---

### **[PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md)** - Navigate the Codebase
Complete directory layout and file organization:
- Directory tree with descriptions
- Build output structure
- How to navigate specific features
- Key dependencies explained
- Storage details (chrome.storage, IndexedDB)
- Common issues & solutions

**Best for**: Finding files, understanding the extension architecture, debugging.

---

## 🚀 Quick Start

1. **For new developers**: Read `PROJECT_DESCRIPTION.md` first for context
2. **To understand components**: Check `COMPONENT_USAGE.md` for the specific component
3. **To navigate codebase**: Use `PROJECT_STRUCTURE.md` as your map
4. **For setup & build**: See `../.github/copilot-instructions.md`

---

## 🔑 Key Concepts

### Data Storage
- **API Keys**: Encrypted in `chrome.storage.local` with optional master password
- **Chat History**: Persisted to `chrome.storage.local` via Zustand
- **Large Content**: Page snapshots & attachments stored in IndexedDB to prevent blocking

### Component Architecture
- **Feature-based**: Each component handles one responsibility
- **Zustand State**: Centralized, persistent state management
- **Markdown UI**: react-markdown + highlight.js for rendering

### Extension Structure (Manifest V3)
- **Background Worker**: Manages tabs and side panel behavior
- **Content Script**: Injects on every page, extracts content
- **Side Panel**: Main React UI (Header → ChatArea → InputArea)

---

## 📁 Related Files

- **Development**: `../.github/copilot-instructions.md` - Build, test, lint commands
- **README**: `../README.md` - User-facing project description (in Chinese)
- **Manifest**: `../manifest.json` - Extension permissions and configuration
- **Config**: `../tsconfig.json`, `../vite.config.ts`, `../tailwind.config.js`

---

## 💡 Common Tasks

### "I want to add a new AI provider"
→ Read: `COMPONENT_USAGE.md` → Settings.tsx section, then `PROJECT_STRUCTURE.md` → Data Flow

### "How do I understand the message flow?"
→ Read: `COMPONENT_USAGE.md` → Header, ChatArea, InputArea sections in order

### "Where is the state stored?"
→ Read: `PROJECT_STRUCTURE.md` → Storage Details, then `COMPONENT_USAGE.md` → Zustand Store

### "How does file upload work?"
→ Read: `COMPONENT_USAGE.md` → InputArea.tsx section

### "What's the encryption strategy?"
→ Read: `PROJECT_DESCRIPTION.md` → 🔒 Security section, then `COMPONENT_USAGE.md` → secureStorage.ts

---

## 🔐 Private/Sensitive Data

This project stores sensitive data locally:
- **API Keys**: Encrypted in `chrome.storage.local`
- **Master Password**: Never transmitted, only used locally for key derivation
- **Chat History**: Local only, not cloud-synced

See `.gitignore` for files excluded from version control (`.env`, `.secrets`, etc.)

---

**Last Updated**: March 29, 2026

For questions or updates to documentation, refer to the relevant component files in `../src/`
