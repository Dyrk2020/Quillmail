# Deadletter

A private, real-time disposable email inbox — right in your browser. Single-page app built on Next.js 16 (App Router) + React 19, talking to pluggable temp-mail backends (DuckMail API by default, Mail.tm switchable) with **Mercure SSE** live delivery. English | [简体中文](README.zh-CN.md)

## Features

- **Multi-account management** — create / switch / delete multiple temporary inboxes; accounts + tokens persist locally
- **Real-time inbox via Mercure SSE** — new mail pushed instantly (`use-mercure-sse`); `/api/sse` route keeps the stream alive with heartbeats, `use-smart-mail-checker` falls back to polling when SSE is unavailable
- **Pluggable providers** — runtime switch between DuckMail (`api.duckmail.local`, default) and Mail.tm (`api.mail.tm`); enable/disable per provider in settings (Mail.tm defaults off, unavailable from Vercel egress IPs)
- **Optional API key** — keyless works on public domains; a key unlocks additional domains and private-domain creation
- **Built-in API proxy** — `/api/mail` route handler proxies all mailbox API calls (provider base-URL injection, 15 s upstream timeout); the browser never talks cross-origin to the backend
- **Bilingual UI (i18n)** — `next-intl` with locale-prefixed routes (`/zh/...`, `/en/...`), browser-language autodetect, `zh` default
- **Dark mode** — `next-themes`; HeroUI + Tailwind CSS + Framer Motion + lucide icons
- **In-app docs** — localized `/[locale]/api-docs`, `/[locale]/faq`, `/[locale]/privacy`

## Stack

| Layer | Choice |
|---|---|
| Framework | Next.js 16 (App Router, Route Handlers) |
| UI runtime | React 19 |
| Styling | Tailwind CSS 3.4 + tailwindcss-animate |
| Components | HeroUI (primary), Radix primitives, lucide-react |
| i18n / theme | next-intl 4 · next-themes |

## Dev & deploy

```bash
npm install
npm run dev        # local dev server
```

Zero-config deploy to Netlify or Vercel (the repo grew out of a Netlify deploy; `netlify.toml`-equivalent settings documented in the zh README).
