# Skill Connect

## Overview

Skill Connect is a professional networking platform for skill exchange. Users can create profiles, list skills they offer or want to learn, connect with other users, and exchange messages. The app matches people who want to teach with those who want to learn, enabling peer-to-peer knowledge sharing.

The application follows a full-stack TypeScript architecture with a React frontend, Express backend, and PostgreSQL database, all deployed as a single unified application on Replit.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend (client/)
- **Framework**: React with TypeScript, bundled by Vite
- **Routing**: Wouter (lightweight alternative to React Router)
- **State Management**: TanStack React Query for server state; no separate client state library
- **UI Components**: shadcn/ui (new-york style) built on Radix UI primitives with Tailwind CSS
- **Styling**: Tailwind CSS with CSS variables for theming (indigo/blue color scheme), custom fonts (Inter for body, Outfit for display)
- **Forms**: React Hook Form with Zod resolvers for validation
- **Path aliases**: `@/` maps to `client/src/`, `@shared/` maps to `shared/`

Key pages: Landing (public), Dashboard, Profile, Network, Messages (all protected via auth check). Protected routes redirect unauthenticated users to the landing page.

Custom hooks in `client/src/hooks/` wrap all API calls using React Query mutations and queries, organized by domain (profiles, skills, connections, messages, auth).

### Backend (server/)
- **Framework**: Express.js on Node with TypeScript (run via tsx in dev, esbuild for production)
- **HTTP Server**: Node's built-in `http.createServer` wrapping Express
- **API Design**: RESTful JSON API under `/api/` prefix. Route definitions and input/output schemas are shared between client and server via `shared/routes.ts`
- **Authentication**: Replit Auth using OpenID Connect (OIDC) with Passport.js. Sessions stored in PostgreSQL via `connect-pg-simple`. Auth logic lives in `server/replit_integrations/auth/`
- **Dev Server**: Vite middleware in development mode with HMR; static file serving in production from `dist/public`

### Shared Code (shared/)
- `shared/schema.ts`: Drizzle ORM table definitions and Zod insert schemas for all entities (users, sessions, profiles, skills, userSkills, connections, messages)
- `shared/models/auth.ts`: User and session table definitions required by Replit Auth (do not modify or drop these tables)
- `shared/routes.ts`: API route contracts with paths, methods, Zod input/output schemas. Used by both client hooks and server route handlers to ensure type safety

### Database
- **Database**: PostgreSQL (required, provisioned via Replit)
- **ORM**: Drizzle ORM with `drizzle-zod` for schema-to-validation integration
- **Schema management**: `drizzle-kit push` (no migration files needed for development; migrations folder exists for production)
- **Connection**: `pg.Pool` using `DATABASE_URL` environment variable
- **Storage pattern**: `DatabaseStorage` class in `server/storage.ts` implements `IStorage` interface, providing a clean abstraction layer over database operations

### Data Model
- **users**: Core user identity (id, email, name, profile image) - managed by Replit Auth
- **sessions**: Session storage for authentication - managed by Replit Auth
- **profiles**: Extended user info (title, bio, location, social links), one-to-one with users
- **skills**: Global skill catalog (unique names)
- **userSkills**: Junction table linking users to skills with type ("offering" or "learning")
- **connections**: Friend/connection requests between users with status (pending/accepted/rejected)
- **messages**: Direct messages between users with read tracking

### Build System
- **Development**: `npm run dev` runs tsx to start the Express server with Vite middleware for HMR
- **Production build**: `npm run build` runs a custom script (`script/build.ts`) that builds the client with Vite and bundles the server with esbuild. Server dependencies on an allowlist are bundled to reduce cold start times
- **Output**: `dist/index.cjs` (server) and `dist/public/` (client static files)

## External Dependencies

### Required Services
- **PostgreSQL Database**: Required. Connection via `DATABASE_URL` environment variable. Used for all data storage including sessions
- **Replit Auth (OIDC)**: Authentication provider. Uses `ISSUER_URL` (defaults to `https://replit.com/oidc`), `REPL_ID`, and `SESSION_SECRET` environment variables

### Key Environment Variables
- `DATABASE_URL` - PostgreSQL connection string (mandatory)
- `SESSION_SECRET` - Secret for signing session cookies (mandatory)
- `REPL_ID` - Replit environment identifier (set automatically by Replit)
- `ISSUER_URL` - OIDC issuer URL (defaults to Replit's OIDC endpoint)

### Notable NPM Packages
- `drizzle-orm` + `drizzle-kit` + `drizzle-zod` - Database ORM and schema tooling
- `express` + `express-session` - HTTP server and session management
- `passport` + `openid-client` - Authentication via OIDC
- `connect-pg-simple` - PostgreSQL session store
- `@tanstack/react-query` - Async server state management
- `wouter` - Client-side routing
- `zod` - Runtime schema validation (shared between client and server)
- `date-fns` - Date formatting
- `lucide-react` - Icon library
- Full shadcn/ui component library (Radix UI primitives)