# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

Open `index.html` directly in a browser — no server or build step required.

If a backend is running at `http://127.0.0.1:3000`, form submissions POST to `/api/name-change`. The UI shows a success screen regardless of whether that request succeeds.

## Architecture

This is a single self-contained file (`index.html`) using React 18 and Tailwind CSS loaded via CDN, with JSX transpiled in-browser by Babel Standalone. There is no build toolchain, no `package.json`, and no node_modules.

**Form sections:**
1. **New Legal Name** — first (required), middle (optional), last (required)
2. **Credit Cards to Update** — add/remove cards by type + last-4 digits; stored in component state as `{ type, lastFour }` objects
3. **Supporting Documents** — separate upload boxes for Driver's License and Passport; at least one required; files are held in state but not uploaded to a server
4. **Declaration checkbox** — must be checked before submit

**Validation** runs on submit only (not on blur). Errors are cleared field-by-field as the user corrects them. The `data-error` attribute on error paragraphs is used as a scroll target.

**Submission** serialises name, cards, and document metadata (filename + type, not binary content) to JSON and POSTs to `/api/name-change`. A success screen with a generated reference ID (`NCR-<base36>`) is shown after submission.

## Key CDN Dependencies

| Library | Version | Role |
|---|---|---|
| React + ReactDOM | 18 | UI rendering |
| Tailwind CSS | v3 (cdn.tailwindcss.com) | Styling |
| Babel Standalone | latest unpkg | In-browser JSX transpilation |
