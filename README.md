<div align="center">

# Quillmail

**A private, real-time disposable email inbox — right in your browser.**

[![Version](https://img.shields.io/badge/version-0.1.0-blue)](./package.json)
[![Next.js](https://img.shields.io/badge/Next.js-16-black?logo=nextdotjs)](https://nextjs.org)
[![React](https://img.shields.io/badge/React-19-61dafb?logo=react)](https://react.dev)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](#license)

[简体中文](./README.zh-CN.md) | English

Quillmail is a single-page temporary email client built on Next.js 16 and React 19.
It talks to pluggable mailbox backends (DuckMail API by default, Mail.tm as an
alternative), receives new messages instantly over Mercure SSE, speaks both
Chinese and English, and deploys to Netlify or Vercel with zero configuration.

**🌐 Live app:** <https://duckmail.local> · **📚 API docs page:** `/en/api-docs`

</div>

<div align="center">
  <img src="./img/display1.png" alt="Quillmail main interface" width="800">
  <p><em>Main interface — clean, modern design</em></p>
  <img src="./img/display2.png" alt="Quillmail email management" width="800">
  <p><em>Email management — messages arrive in real time</em></p>
</div>

## ✨ Features

- 📧 **Multi-account management** — create, switch between, and delete multiple temporary mailboxes; accounts and tokens are persisted locally per account.
- 🔄 **Real-time inbox over Mercure SSE** — new messages are pushed the moment they arrive via `use-mercure-sse`; a built-in `/api/sse` route keeps the connection alive with heartbeats, and a smart checker (`use-smart-mail-checker`) falls back to polling when SSE is unavailable.
- 🔧 **Multiple API providers** — switch between the DuckMail API (`api.duckmail.local`, default) and Mail.tm (`api.mail.tm`) at runtime; providers can be enabled/disabled in settings (Mail.tm ships disabled by default and does not work from Vercel IPs).
- 🔑 **Optional API Key** — without a key you get public domains with full basic functionality; with a key you unlock more domains and private-domain creation (configure it in the in-app settings panel).
- 🌐 **Bilingual UI (i18n)** — Chinese and English via `next-intl`, with locale-prefixed routes (`/zh/...`, `/en/...`) and automatic browser-language detection; default locale is `zh`.
- 🌙 **Dark mode** — light/dark theme switching powered by `next-themes`.
- 🎨 **Modern UI** — HeroUI components on Tailwind CSS, with Framer Motion animations and lucide icons.
- 🛡️ **Built-in API proxy** — a Next.js route handler (`/api/mail`) relays all mailbox API calls, injecting the provider base URL and tolerating slow upstreams (15 s timeout), so the browser never talks cross-origin to the backend directly.
- 📖 **In-app docs** — localized API documentation, FAQ, and privacy pages at `/[locale]/api-docs`, `/[locale]/faq`, and `/[locale]/privacy`.

## 🧱 Tech Stack

| Layer | Choice |
| --- | --- |
| Framework | [Next.js](https://nextjs.org) 16 (App Router, Route Handlers) |
| UI runtime | [React](https://react.dev) 19 |
| Styling | Tailwind CSS 3.4 + tailwindcss-animate |
| Components | [HeroUI](https://heroui.com) (primary), Radix UI primitives, lucide-react icons |
| i18n | [next-intl](https://next-intl.dev) 4 (`zh` / `en`) |
| Theming | [next-themes](https://github.com/pacocoursey/next-themes) |
| Real-time | Mercure hub + native `EventSource` / `@microsoft/fetch-event-source` |
| Forms & validation | react-hook-form + zod |
| Language | TypeScript 5 |

> Requires **Node.js ≥ 20.9** and **pnpm**.

## 🚀 Quick Start

```bash
# 1. Install dependencies
pnpm install

# 2. Start the dev server
pnpm dev

# 3. Open http://localhost:3000 (redirects to /zh or /en based on your browser)
```

Production build:

```bash
pnpm build
pnpm start
```

No environment variables are required — provider endpoints (DuckMail, Mail.tm,
Mercure hub) are pre-configured in `lib/api.ts` and can be switched in the app's
settings panel.

## 🔌 API Providers

Quillmail is provider-agnostic: a provider is just a `{ id, name, baseUrl, mercureUrl }`
config resolved in `lib/api.ts`.

- **DuckMail** (default) — self-hosted mailbox backend. All endpoints work
  without a key; account creation returns a token used to authenticate
  mailbox-scoped operations. The `domains` and `accounts` endpoints additionally
  accept an API Key header, which unlocks private domains for that key.
- **Mail.tm** — the well-known public disposable-email API. Disabled by default
  (enable it in settings); note that Mail.tm blocks Vercel egress IPs, so it
  will not work on Vercel deployments.

Provider behavior highlights:

- Requests are proxied through `/api/mail`, which reads an
  `X-API-Provider-Base-URL` header to route to the selected provider.
- Rate limit: 12 QPS. Email retention: 3 days, then automatic deletion.
- Accounts created via the API accept an `expiresIn` parameter (seconds);
  `0` / `-1` = never expires, omitted = auto-cleanup after 24 hours.
  Accounts created through the web UI never expire by default.
- There is no password recovery — a temporary inbox is disposable by design.

## ☁️ Deployment

### Netlify (recommended)

A ready-made [`netlify.toml`](./netlify.toml) pins Node 20.9.0 and builds with pnpm —
click the button and Netlify forks and deploys the project automatically:

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/Dyrk2020/Quillmail)

### Vercel

Vercel auto-detects the Next.js project; no extra configuration needed:

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/Dyrk2020/Quillmail)

> ⚠️ On Vercel, keep the Mail.tm provider disabled (its IPs are blocked by
> Mail.tm); the DuckMail provider works out of the box.

## 📁 Project Structure

```
Quillmail/
├── app/
│   ├── [locale]/            # Localized pages: home, api-docs, faq, privacy
│   └── api/
│       ├── mail/            # Proxy route → selected mailbox API provider
│       └── sse/             # SSE relay with heartbeat keep-alive
├── components/              # UI: header, sidebar, message list/detail, settings…
│   └── ui/                  # Base UI primitives (shadcn-style)
├── contexts/                # API-provider, auth, and mail-status contexts
├── hooks/                   # Mercure SSE, smart mail checker, toasts, mobile
├── i18n/                    # next-intl routing / request / navigation config
├── lib/                     # API client (providers, accounts, messages), utils
├── messages/                # Translation catalogs: en.json, zh.json
├── types/                   # Shared TypeScript interfaces (Domain, Account, Message)
├── middleware.ts            # next-intl locale routing middleware
├── netlify.toml             # Netlify build config (Node 20.9, pnpm build)
└── next.config.mjs          # next-intl plugin, image optimization off
```

## 📄 License

Released under the [MIT License](LICENSE).

## ☕ Support

If Quillmail saves you time, a star is always appreciated — and if you'd like
to help cover backend costs:

[![爱发电](https://img.shields.io/badge/%E7%88%B1%E5%8F%91%E7%94%B5-syferie-946ce6?style=for-the-badge)](https://afdian.com/a/syferie)

## 📮 Contact

Questions or suggestions? Open an
[issue](https://github.com/Dyrk2020/Quillmail/issues) or email
[syferie@proton.me](mailto:syferie@proton.me).