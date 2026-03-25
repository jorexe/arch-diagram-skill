# arch-diagram

![Dark mode](https://github.com/jorexe/arch-diagram-skill/blob/gh-pages/example/architecture-diagram-dark.png?raw=true)

A Claude Code skill that analyzes your codebase and generates an interactive, polished system architecture diagram as a standalone HTML file.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Donate](https://img.shields.io/badge/Donate-PayPal-blue.svg)](https://www.paypal.com/donate/?hosted_button_id=VDQK9WM46RAME)

## What it does

Point it at any codebase and it will:

1. **Detect** your services, external providers, databases, and how they connect
2. **Confirm** findings with you before generating anything
3. **Generate** a single self-contained HTML file with an interactive architecture diagram

The output is a clean, infographic-style diagram — not a generic box-and-arrow chart.

### Features

- **Auto-detection** — Scans Nx monorepos, Docker Compose, K8s manifests, or single apps
- **External providers** — Finds AI providers, databases, auth, monitoring, payments, and more from imports and env vars
- **Color-coded arrows** — Connections between services are categorized and color-coded with a legend
- **Dark mode** — Toggle between light and dark themes (persists to localStorage)
- **Pan & zoom** — Mouse drag to pan, scroll to zoom, double-click to reset
- **Touch support** — Works on tablets with pinch-to-zoom
- **Zero dependencies** — Single HTML file, no CDNs, works with `file://`

### Live example

Open [`live demo`](https://jorexe.github.io/arch-diagram-skill) in your browser to see a sample diagram with pan, zoom, dark mode, and color-coded arrows.

### Screenshots

![Light mode](https://github.com/jorexe/arch-diagram-skill/blob/gh-pages/example/architecture-diagram-bright.png?raw=true)

![Dark mode](https://github.com/jorexe/arch-diagram-skill/blob/gh-pages/example/architecture-diagram-dark.png?raw=true)

## Install

```bash
claude install-skill https://github.com/jorexe/arch-diagram-skill
```

## Usage

Just ask Claude to diagram your architecture:

```
> /arch-diagram
> diagram my services
> show me how my system connects
> create an architecture infographic
> draw my architecture
```

Claude will analyze your codebase, show you what it found, and generate `architecture-diagram.html` in your project root.

### Iteration

After generating, you can ask for changes:

- "Add Redis to the diagram"
- "Move the frontend group to the left"
- "Change the arrow colors"
- "Remove the Order Service"

Claude reads the existing file before editing, so your customizations are preserved.

## What gets detected

| Category | Examples |
|---|---|
| **Services** | Nx apps, Docker Compose services, K8s deployments |
| **AI providers** | OpenAI, Anthropic, Groq, Replicate, HuggingFace, Cohere, Google GenAI |
| **Databases** | MongoDB, PostgreSQL, MySQL, Redis, Elasticsearch |
| **Auth** | Firebase, Auth0, Clerk, Supabase |
| **Payments** | Stripe, PayPal, Square |
| **Monitoring** | Sentry, Datadog, New Relic, Grafana |
| **Cloud** | AWS, GCP, Azure, Cloudflare, Vercel, Railway |
| **Email** | SendGrid, Resend, Twilio, Postmark |
| **Analytics** | Google Analytics, Plausible, Mixpanel, PostHog |
| **Ads** | Google AdSense, AdMob |

## How it works

The skill includes a `template.html` with all the boilerplate (CSS variables for light/dark themes, pan/zoom JS, SVG arrow engine). When generating a diagram, Claude reads the template and fills in project-specific content:

- Service cards with icons, descriptions, and badges
- Connection definitions using the `connect()` arrow API
- Legend matching the project's connection types
- Footer badges summarizing the stack

This means consistent quality across generations — the interactive features don't get regenerated from scratch each time.

## Support

If you find this skill useful, consider buying me a coffee:

[![Donate](https://img.shields.io/badge/Donate-PayPal-blue.svg)](https://www.paypal.com/donate/?hosted_button_id=VDQK9WM46RAME)

## License

[MIT](LICENSE)
