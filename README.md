# NETRA

NETRA is a modern intelligence and case-management platform built with Next.js for investigating criminal networks, linking entities, analyzing evidence, and coordinating case workflows across investigators and administrators.

At a high level, the application combines:
- a case workspace for creating and tracking investigations,
- AI-powered FIR / case analysis integrations,
- MongoDB-backed persistence for case records and activity logs,
- a dark, operational dashboard experience for security and intelligence teams.

The current codebase is centered around a Next.js App Router application, with route handlers in `app/api/*` that bridge local MongoDB data and an external FIR intelligence API.

## Why this project exists

NETRA is designed for teams that need to:
- register new investigations quickly,
- attach incoming case sources and documents,
- orchestrate AI-assisted document extraction and relationship analysis,
- visualize entities, networks, and timeline events,
- review case activity and operational status from one workspace.

This is not a generic blog or starter app — it is a workflow-oriented investigation dashboard for criminal intelligence analysis.

## Stack

- Frontend: Next.js 16, React 19, TypeScript
- Styling: Tailwind CSS
- Runtime: Node.js 22
- Data layer: MongoDB via Mongoose
- UI/visual layers: custom motion effects, 3D globe visuals, custom graph/timeline components
- AI integration: external FIR intelligence API (`lib/aiApi.ts`)
- Containerization: Docker and Docker Compose

### Core dependencies

Key packages that shape the application behavior:
- `next` — App Router framework and server routes
- `react`, `react-dom` — UI rendering
- `mongoose` — MongoDB model layer
- `@supabase/supabase-js` — present in project dependencies and likely intended for auth/data integration
- `cytoscape` — network graph visualization
- `three` — globe / visual display layer
- `motion` — animation effects for the dashboard and landing experience

## Architecture overview

### Client-facing application

The app is structured as a Next.js App Router SPA-like experience:
- `app/page.tsx` — landing page / portal entry experience
- `app/login/page.tsx` — role selection (admin vs investigator)
- `app/investigator/dashboard/page.tsx` — investigator workspace overview
- `app/admin/dashboard/page.tsx` — admin dashboard workbench
- `app/cases/new/page.tsx` — creating a case record
- `app/cases/[caseCode]/...` — per-case investigation pages

### API and server layer

API routes live under `app/api` and handle both local application logic and AI integration:
- `app/api/cases/route.ts` — list/create cases
- `app/api/cases/[caseCode]/route.ts` — fetch/update a specific case
- `app/api/activities/route.ts` — recent activity feed
- `app/api/ai/*` — AI analysis, chat, graph, summary, timeline, linking, upload endpoints
- `app/api/ai/case/route.ts`, `app/api/ai/summary/route.ts`, `app/api/ai/chat/route.ts` and related routes provide the system’s operational intelligence layer

### Data model layer

- `models/case.ts` — MongoDB schema for investigation records, priority, status, sources, and AI linkage
- `models/activity.ts` — activity feed records for case events
- `lib/mongodb.ts` — MongoDB connection helper with singleton connection caching
- `lib/auth.ts` — demo role-based auth helpers for admin/investigator login state

### Integration layer

The project does not keep all intelligence logic locally; it talks to an external FIR API service configured through `lib/aiApi.ts`.

This layer exposes wrappers for:
- fetching case lists and details,
- creating FIR cases,
- uploading documents,
- checking job status,
- summarizing cases,
- graphing entities/relationships,
- analyzing investigations,
- retrieving chat history and timelines,
- linking a local case to an AI-backed case record.

In practice, local MongoDB functions as the operational state store, while the FIR backend provides the intelligence and extraction pipelines.

## Project structure

```text
.
├── app/                         # Next.js App Router frontend and API routes
│   ├── admin/                  # admin views
│   ├── api/                    # route handlers for cases, AI, and activity APIs
│   ├── cases/                  # case detail, timeline, and creation flows
│   ├── investigator/           # investigator dashboard and workspace
│   ├── globals.css             # global styles and theme
│   ├── layout.tsx              # root layout and chatbot injection
│   ├── login/                  # login flow
│   └── page.tsx                # landing page
├── components/                 # UI widgets, panels, network map, dossier, chat, etc.
├── lib/                        # app utilities and domain integrations
│   ├── aiApi.ts                # FIR AI API client
│   ├── auth.ts                 # role/session helpers
│   ├── mongodb.ts              # MongoDB singleton connection
│   ├── workspaceCase.ts        # case load + live sync helpers
│   └── ...
├── models/                     # Mongoose schemas
├── public/                     # static assets
├── test-files/                 # sample or test input files
├── types/                      # shared types
├── .dockerignore
├── .gitignore
├── AGENTS.md
├── CLAUDE.md
├── Dockerfile
├── Dockerfile.dev
├── docker-compose.yml
├── docker-compose.dev.yml
├── eslint.config.mjs
├── next.config.ts
├── package.json
├── tsconfig.json
├── README.md
└── ...
```

## How the system fits together

The main runtime flow is:

1. A user logs in as `admin` or `investigator` using the mocked credentials exposed in `app/login/page.tsx`.
2. The application loads and displays cases from MongoDB through API routes.
3. When creating a case, the app attempts to resolve or create a matching AI case ID in the external FIR service.
4. Case records are enriched with AI-backed metadata, extracted entities, timeline data, analysis, and relationship graph results.
5. The UI renders operational dashboards, entity panels, network maps, timeline views, and activity logs from both local and AI-sourced data.

This makes the project a hybrid system: local operational workflow + external intelligence service + rich frontend experience.

## Key features

- Case registration and workspace management
- Role-based access for admin and investigator flows
- AI-driven FIR analysis and extraction pipeline
- Network/entity relationship mapping
- Timeline and activity tracking
- Document upload and case source ingestion
- Dashboard metrics for active, high-priority, and reviewed cases
- Dark-themed investigation interface with operational visuals

## Environment variables

Create a `.env.local` file in the project root before running the app.

```bash
MONGODB_URI=mongodb://localhost:27017/netra
NEXT_PUBLIC_AI_API_URL=https://fir-intelligence-api.onrender.com/api/v1
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=your_supabase_key
```

Notes:
- `MONGODB_URI` is required for the app to connect to MongoDB.
- `NEXT_PUBLIC_AI_API_URL` is optional because the code defaults to the Render FIR API endpoint.
- `NEXT_PUBLIC_SUPABASE_*` settings are part of the current app config and may be used by related features or future integrations.

## Local development

### Install dependencies

```bash
npm install
```

### Run the application

```bash
npm run dev
```

Then open:
- http://localhost:3000

### Build for production

```bash
npm run build
npm run start
```

### Linting

```bash
npm run lint
```

## Docker

### Run with Docker Compose

```bash
docker compose up --build
```

The project also includes `Dockerfile.dev` and `docker-compose.dev.yml` for development-oriented container workflows.

## Authentication notes

The current repository uses a lightweight local-storage-based role concept:
- `admin` → `netra-admin` / `netra@2026`
- `investigator` → `netra-investigator` / `investigator@2026`

This is a convenience pattern for demo and internal use, but it is not production-grade authentication. For a real production deployment, replace this with secure server-side sessions or a proper identity provider (for example Supabase Auth, Auth.js, or a custom JWT/session model).

## Production considerations

Before deploying this app beyond a prototype or internal system, consider:

- replacing localStorage auth with secure backend session management,
- moving AI provider credentials and sensitive values to server-side environment variables,
- validating and sanitizing uploaded source content,
- adding rate limiting and request throttling to public API endpoints,
- implementing user permissions and audit logging beyond the current role model,
- adding automated tests for API routes and core case flows,
- configuring production MongoDB, Supabase, and monitoring/alerting pipelines.

## Recommended next steps

1. Add a proper `.env.example` file for reproducible onboarding.
2. Replace demo auth with secure authentication.
3. Add unit/integration tests for database and case API flows.
4. Add API documentation for the `app/api/ai/*` endpoints.
5. Harden the document ingestion pipeline and validation layers.
6. Add CI/CD checks for linting, build, and test gates.

## Summary

NETRA is a case-intelligence platform with a strong operational UX and a hybrid architecture: MongoDB for app state, a Next.js frontend for the experience, and an external FIR intelligence API for extraction and analysis. It is best understood as a security investigation workspace designed to make large, messy case data legible and actionable.

This repository is already structured for real internal use, but it still benefits from stronger authentication, environment management, and production-hardening before a public or enterprise deployment.

## Contributing

Use the project as a Node.js/Next.js app with TypeScript:

```bash
npm install
npm run dev
```

If you are working on route handlers, AI integrations, or case workflows, start from the relevant files under `app/api`, `lib`, and `models`.

## License

This repository does not currently declare a license in the project metadata. If you plan to publish or distribute it externally, add an explicit license file and document the intended usage rights.
