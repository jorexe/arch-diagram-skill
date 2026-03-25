---
name: arch-diagram
description: "Analyze a codebase and generate an interactive, polished system architecture diagram as a standalone HTML file. Use this skill whenever the user asks to visualize their architecture, create a system diagram, map out their services, show how their backend is structured, or wants an overview of their system's components and external providers. Also trigger when the user says things like 'draw my architecture', 'diagram my services', 'show me how my system connects', or 'create an infographic of my stack'. This skill produces a beautiful, interactive HTML diagram — not a generic box-and-arrow chart."
---

# Architecture Diagram Generator

Generate interactive, visually polished system architecture diagrams by analyzing a codebase. The output is a standalone HTML file with pan, zoom, and a clean infographic aesthetic — not a generic box-and-arrow tool diagram.

## What this skill produces

A single self-contained HTML file that shows:
- **Services** as styled cards grouped by role (backend, frontend, orchestration, etc.)
- **External providers** (AI models, cloud services, payment processors, etc.)
- **Databases and data stores**
- **Arrows** showing communication flow between groups
- **Badges/tags** with key metadata (port numbers, protocols, tech stack)

The diagram is interactive: users can pan, zoom, and toggle dark mode.

## How to analyze the codebase

The goal is to build a mental model of the system before generating any HTML. Scan in this order — stop early if you have enough signal from the first few steps.

### Step 1: Detect project structure

Look for monorepo indicators first, then fall back to single-app patterns:

- **Nx monorepo**: Check `nx.json`, `workspace.json`, or `project.json` files. Scan `apps/` for services and `libs/` for shared libraries (but don't diagram libraries — just note their existence).
- **Docker Compose**: `docker-compose.yml` or `compose.yml` — each service block is likely a diagrammable component.
- **Package.json workspaces**: `workspaces` field in root `package.json`.
- **Kubernetes**: `k8s/`, `helm/`, or `manifests/` directories.
- **Single app**: If none of the above, treat the root as one service and focus on its external connections.

### Step 2: Identify services and their roles

For each service/app found, determine:
- **Name** (use the directory or config name)
- **Type**: backend API, frontend app, worker, gateway/proxy, orchestrator, microservice
- **Port** (from config files, Dockerfiles, or environment variables)
- **Tech stack** (framework, language — from package.json, go.mod, requirements.txt, Cargo.toml, etc.)
- **Communication protocol** with other services (HTTP, gRPC, WebSocket, message queue, SSE)

### Step 3: Detect external providers

Search for these signals across the codebase:

- **AI/ML providers**: imports or env vars referencing OpenAI, Anthropic, Groq, Replicate, HuggingFace, Cohere, etc.
- **Cloud services**: AWS SDK, GCP client libs, Azure SDK, Cloudflare, Vercel, Railway
- **Databases**: MongoDB, PostgreSQL, MySQL, Redis, Elasticsearch connection strings or drivers
- **Auth providers**: Firebase, Auth0, Clerk, Supabase Auth
- **Payment**: Stripe, PayPal, Square SDKs
- **Email/messaging**: SendGrid, Resend, Twilio, Postmark
- **Monitoring**: Sentry, Datadog, New Relic, Grafana
- **Storage**: S3, GCS, Cloudinary, Uploadthing
- **Analytics**: Google Analytics, Plausible, Mixpanel, PostHog
- **Ads**: Google AdSense, AdMob

Look in: `package.json` dependencies, import statements, `.env` / `.env.example` files, config modules, and docker-compose service definitions.

### Step 4: Map connections

Determine how services talk to each other:
- Gateway/proxy → backend services (HTTP routes, proxy config)
- Service-to-service calls (HTTP clients, gRPC stubs, message queue producers/consumers)
- Frontend → backend (API base URL, SSE/WebSocket endpoints)
- Services → external providers (SDK usage, API clients)
- Services → databases (connection modules, ORMs)

### Step 5: Confirm with the user

Before generating, present a summary:

```
I found the following architecture:

**Services:**
- Gateway (NestJS, port 3000) — proxies to all backend services
- Player Service (NestJS, port 3001) — character management
- ...

**External Providers:**
- Groq (AI, llama-3.3-70b-versatile)
- MongoDB (database)
- Firebase (auth)
- ...

**Connections:**
- Client → Gateway → [Player, World, Quest, Game Orchestrator]
- Game Orchestrator → AI Service → Groq
- ...

Does this look right? Anything to add, remove, or rename?
```

Wait for confirmation before generating the HTML.

## HTML Generation

### Use the template

Read the template at `template.html` (bundled with this skill). It provides all the boilerplate:
- Light/dark theme with CSS variables and toggle switch
- Pan + zoom (mouse, touch, trackpad)
- SVG arrow engine with `connect()` helper
- Card, badge, group, and layout CSS classes

**Do NOT regenerate the CSS, JS, theme toggle, or pan/zoom from scratch.** Copy the template and fill in the placeholder sections.

### Template placeholders

The template has clearly marked `{{PLACEHOLDER}}` comments with examples. Fill in each:

| Placeholder | What goes there |
|---|---|
| `{{TITLE}}` | Project name (appears in `<title>` and diagram header) |
| `{{LEGEND}}` | Legend div with color-coded connection types for this project |
| `{{CLOUD_SERVICES}}` | Top row of compact cards for external providers (Firebase, Sentry, etc.) |
| `{{LEFT_REGION}}` | Left column — typically AI providers, external APIs, or secondary services |
| `{{CENTER_REGION}}` | Center column — core backend services in a group with `grid-2col` |
| `{{RIGHT_REGION}}` | Right column — frontend apps, deployment targets |
| `{{BOTTOM_REGION}}` | Bottom row — databases, data stores |
| `{{FOOTER}}` | Footer badges summarizing the stack |
| `{{CONNECTIONS}}` | JS `connect()` calls defining arrows between `data-id` elements |

### Layout strategy

Arrange components spatially by role. The template's default 3-column grid works for most architectures:

```
    [External Cloud Services]        (top row, region-cloud)

[AI/External]   [Backend Services]   [Frontend Apps]
  (left)           (center)            (right)

              [Databases / Data Stores]
                   (bottom)
```

Adapt if needed — if there's no left column, change `grid-template-columns` to `1fr 320px`. If there are 4 columns, add a `region-far-right`.

### Card structure

Every service card needs a `data-id` attribute for the arrow engine to target it:

```html
<div class="card" data-id="my-service">
  <div class="card-icon">🎯</div>
  <div class="card-body">
    <div class="card-name">My Service</div>
    <div class="card-desc">What it does</div>
    <div class="card-badges">
      <span class="badge badge-blue">Port 3000</span>
    </div>
  </div>
</div>
```

Available CSS helpers:
- `grid-2col` — 2-column grid inside a group
- `span-full` — card spans both columns in a `grid-2col`
- `chain` + `chain-arrow` — vertical chain with "↓ fallback" labels
- `card-cloud` — compact card for the top cloud services row
- Badge colors: `badge-green`, `badge-blue`, `badge-purple`, `badge-orange`, `badge-red`, `badge-gray`, `badge-teal`

### Connection definitions

In the `{{CONNECTIONS}}` section of the template JS, define arrows using `connect()`:

```javascript
const dark = document.documentElement.classList.contains('dark');
const PROXY  = dark ? '#64748B' : '#94A3B8';   // gray
const ORCH   = dark ? '#F59E0B' : '#E67E22';   // orange
const AI     = dark ? '#C4B5FD' : '#A78BFA';   // purple
const DB     = dark ? '#64748B' : '#94A3B8';   // gray
const CLIENT = dark ? '#34D399' : '#10B981';   // green

connect('client', 'gateway', 'left', 'right', { color: CLIENT, width: 2, label: 'REST', curve: -0.08 });
connect('gateway', 'player', 'bottom', 'top', { color: PROXY, width: 1.5 });
```

**Sides**: `left`, `right`, `top`, `bottom`, `top-left`, `top-right`, `bottom-left`, `bottom-right`
**Options**: `color`, `width`, `dash` (bool), `label` (string), `curve` (float, negative curves the other way)

Choose 3-5 arrow color categories that match the project's connection types, and add a matching legend.

## Output

Save the HTML file to the project root as `architecture-diagram.html` (or ask the user where they want it). After saving, tell the user:

> "I've saved the architecture diagram to `architecture-diagram.html`. Open it in your browser — you can pan by dragging and zoom with scroll/pinch. Use the toggle in the top-left for dark mode. Let me know if you'd like any changes."

## Iteration

When the user asks for changes:
- **Content changes** (add/remove/rename a service): Update the relevant card and connections
- **Layout changes** (move a group, change flow direction): Adjust the CSS grid or region positions
- **Style changes** (colors, fonts, sizes): Update the CSS variables
- **Export**: If the user wants a PNG/SVG, suggest browser screenshot or print-to-PDF

Read the current `architecture-diagram.html` before making edits to avoid overwriting prior customizations.
