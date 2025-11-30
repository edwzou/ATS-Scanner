# ATS Scanner

AI-assisted resume analysis for Applicant Tracking Systems (ATS). Candidates can upload a PDF resume, attach job details, and instantly receive structured feedback that scores ATS alignment, tone, content quality, structure, and skills. All uploads, generated screenshots, and AI responses are persisted via Puter.js so users can revisit detailed reports later.

## Features

- 🔐 **Puter.js auth + storage** – leverage Puter’s hosted auth, KV, and filesystem APIs to keep each user’s uploads and feedback scoped to their own account.
- 🖼 **PDF ➜ PNG preview** – browser-side pdf.js pipeline converts the first page of the uploaded resume into a high-resolution image that is saved alongside the PDF for quick previews.
- 🧠 **Claude-powered feedback** – dynamically generated prompts (company, role, description) are fed into Puter’s Claude Sonnet integration to produce JSON feedback in a consistent schema.
- 📊 **Rich UI components** – reusable cards, score gauges, accordions, and detail sections render ATS scores, improvement tips, and status indicators across the experience.
- ⚡ **React Router full-stack** – SSR-ready routing with `home`, `upload`, `resume`, `auth`, and utility routes, plus HMR during development.

## Tech Stack

- React 19 + React Router 7
- TypeScript, Tailwind CSS
- Puter.js SDK (auth, fs, ai, kv)
- pdf.js `legacy` build for in-browser conversion
- Vite + @react-router/dev tooling

## Getting Started

1. **Install dependencies**
   ```bash
   npm install
   ```
2. **Configure Puter.js**
   - Ensure the Puter browser extension / SDK is available (the app relies on `window.puter` being injected).
   - Sign in via the in-app auth flow before uploading resumes.
3. **Run the dev server**
   ```bash
   npm run dev
   ```
   The app runs at `http://localhost:5173` with HMR and React Router’s dev server.

## Key Scripts

| Script            | Purpose                               |
|-------------------|---------------------------------------|
| `npm run dev`     | React Router dev server with HMR      |
| `npm run build`   | Generate production client + server   |
| `npm run start`   | Serve the built output (`npm run build` first) |
| `npm run typecheck` | Generate route types + run `tsc`     |

## Architecture Overview

- `app/routes/*` – Top-level pages (`home`, `upload`, `resume`, `auth`, etc.).
- `app/components/*` – UI primitives for score displays, accordions, résumé cards, uploader, etc.
- `app/lib/pdf2img.ts` – pdf.js integration that loads the legacy worker bundle at runtime and converts PDFs to PNG.
- `app/lib/puter.ts` – Centralized Zustand store exposing `auth`, `fs`, `ai`, and `kv` helpers wrapping `window.puter`.
- `constants/index.ts` – Seed data plus prompt template helpers for the AI response.

## Authentication & Authorization

The application uses **Puter.js** for authentication and authorization, providing a seamless, cloud-based auth system without requiring a separate backend.

### Authentication Flow

1. **Initialization**: On app load (`app/root.tsx`), the Puter.js SDK is injected via a script tag, and the Zustand store automatically checks authentication status using `checkAuthStatus()`.

2. **Sign In/Out**: Users interact with the `/auth` route:
   - **Sign In**: Calls `puter.auth.signIn()`, which opens Puter's authentication modal (supports various OAuth providers)
   - **Sign Out**: Calls `puter.auth.signOut()` to clear the session
   - **Auto-redirect**: After successful authentication, users are redirected to their intended destination via the `next` query parameter (e.g., `/auth?next=/resume/123`)

3. **Protected Routes**: Routes are protected at the component level:
   - **Home page** (`/`): Redirects to `/auth?next=/` if not authenticated
   - **Resume detail** (`/resume/:id`): Redirects to `/auth?next=/resume/:id` if not authenticated
   - Other routes check authentication status before allowing access to sensitive operations

4. **State Management**: The `usePuterStore` Zustand store maintains:
   - `auth.isAuthenticated`: Boolean flag indicating authentication status
   - `auth.user`: Current user object with username and profile information
   - `auth.signIn()` / `auth.signOut()`: Methods to manage authentication
   - `auth.checkAuthStatus()`: Verifies current session validity
   - `isLoading`: Loading state during auth operations

### Data Isolation

Once authenticated, all user data is automatically scoped to their Puter account:
- **Resume uploads**: Stored in the user's Puter filesystem (`fs.upload()`)
- **Resume metadata**: Saved in Puter KV store with keys like `resume:{uuid}`
- **AI feedback**: Linked to the authenticated user's account via Puter's AI service

This ensures complete data privacy and multi-user support without additional backend logic.

## Deployment Notes

1. Build the project:
   ```bash
   npm run build
   ```
2. Deploy both `build/client` (static assets) and `build/server` (Node handler) to your hosting of choice. The included Dockerfile bundles everything for container deployments.
3. Because the runtime expects `window.puter`, host the app in an environment where the Puter SDK is injected (e.g., via their browser extension or embedding in the Puter platform).

## Troubleshooting

- **PDF conversion errors**: Ensure the pdf.js worker file is served (Vite handles this via the dynamic import). Browser console logs include detailed reasons when conversion fails.
- **AI feedback failures**: Confirm you are authenticated with Puter and have access to the Claude Sonnet model within your account.
- **Missing assets**: Images and PDF icons live under `public/`; ensure build tooling copies these assets.

---

Built with ❤️ to help candidates understand how their resumes perform against modern ATS systems.
