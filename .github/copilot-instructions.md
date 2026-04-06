# Copilot Instructions for LuminaSider

LuminaSider is a Manifest V3 Chrome extension built with React, TypeScript, and Vite. It provides an AI side panel with optional page-context extraction.

## Build and Validate

Use these commands for agent work:

```bash
npm install
npm run dev
npm run build
npm run preview
npm run pack
```

- `npm run build` runs strict TypeScript checks (`tsc`) before Vite build.
- There are currently no dedicated test or lint scripts.

## Architecture

- `src/background/index.ts`: Manifest V3 service worker for side panel setup, tab lifecycle, and cross-context messaging.
- `src/content/index.ts`: Content script that extracts readable page content via Mozilla Readability.
- `src/App.tsx` + `src/components/*`: Side panel React UI.
- `src/store/index.ts`: Zustand state with persistence and app-level orchestration.

Core messaging flow:
- Background sends tab-change events (`{ action: 'TAB_CHANGED', tabId }`).
- UI requests extraction from content script using async `chrome.runtime.sendMessage`.

## Data and Storage Conventions

- Use `chrome.storage.local` for small, fast settings and metadata.
- Use IndexedDB (`idb-keyval`) for large payloads (page snapshots, attachment blobs).
- Keep the split strict: metadata in Zustand/`chrome.storage.local`, heavy content in IndexedDB.
- Do not introduce direct `localStorage` usage.

## Coding Conventions

- TypeScript strict mode is required; avoid `any`.
- Keep files in `.ts`/`.tsx` only.
- Use Tailwind + existing design tokens/utilities; keep icon usage in `lucide-react`.
- Prefer self-contained feature components in `src/components`.
- Avoid direct DOM manipulation in UI code; let React own rendering.

## Extension-Specific Gotchas

- Manifest V3 only: avoid MV2 patterns.
- Always URL-filter before extraction/messaging (exclude `chrome://`, `edge://`, similar browser internal pages).
- Keep Chrome messaging async-safe; if a listener responds asynchronously, return `true` from the listener.
- Revoke temporary blob URLs after use to prevent leaks.

## API and Provider Notes

- Settings support Google Gemini and OpenAI-compatible providers.
- Keep provider/model/base URL configuration centralized in `src/components/Settings.tsx` and persisted via store actions.

## Documentation (Link, Don’t Embed)

- `README.md`: user-facing setup and usage.
- `docs/README.md`: docs hub.
- `docs/PROJECT_DESCRIPTION.md`: architecture and data-flow details.
- `docs/PROJECT_STRUCTURE.md`: directory/file map.
- `docs/COMPONENT_USAGE.md`: component-level behavior and patterns.
- `docs/QUICK_REFERENCE.md`: quick implementation notes and caveats.
- `docs/plans/`: historical design notes for delivered features.

Prefer updating those docs for deep detail instead of expanding this file.

## MCP

- Playwright MCP is available for browser-driven verification scenarios.
