# Portal Ferramentas — Domain Migration Design

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Migrate portal from `trabalho.deyvyd.com` to `ferramentas.deyvyd.com`, rename repo, move docgen to its own subdomain `docgen.ferramentas.deyvyd.com` on Railway (removing the broken Vercel proxy approach).

**Architecture:** Two separate deployments — Vercel serves the static portal at `ferramentas.deyvyd.com`, Railway serves the Flask+Vue docgen app at `docgen.ferramentas.deyvyd.com`. No proxy between them. DNS wired via two CNAME records.

**Tech Stack:** Vercel static hosting, Railway (Flask+Vue), DNS CNAME records, git revert.

---

## Domain Structure

```
ferramentas.deyvyd.com                              → Vercel (portal-ferramentas repo)
ferramentas.deyvyd.com/aiyra/                       → portal landing
ferramentas.deyvyd.com/aiyra/gerador-relatorio-sustentacao
ferramentas.deyvyd.com/aiyra/asap-chamados
ferramentas.deyvyd.com/aiyra/asap-devsecops
docgen.ferramentas.deyvyd.com                       → Railway (gerador-de-documentacoes repo)
```

## Repo: portal-ferramentas (renamed from portal-trabalho)

### Files to modify

- `vercel.json` — remove all Railway rewrites; keep only `/` → `/aiyra/` redirect
- `aiyra/index.html` — nav brand, title, meta: `ferramentas · deyvyd moura`; docgen card href → `https://docgen.ferramentas.deyvyd.com`
- `aiyra/gerador-relatorio-sustentacao/index.html` — update any domain refs
- `aiyra/asap-chamados/index.html` — update any domain refs
- `aiyra/asap-devsecops/index.html` — update any domain refs

### vercel.json after migration

```json
{
  "redirects": [
    { "source": "/", "destination": "/aiyra/", "permanent": true }
  ]
}
```

## Repo: gerador-de-documentacoes

Revert the 3 commits added during the failed Vercel proxy attempt:

- `edb6dec` — rewrite requirements.txt as UTF-8, add Flask-Cors
- `c5582fd` — add Flask-CORS, route API calls via VITE_API_URL env var
- `26076b0` — set base path /aiyra/docgen/ for Vercel proxy

Target state: `5d35bf0` (atualizando documentos de desenvolvimento) — no flask-cors, no VITE_API_URL, no Vite base path, no Vue Router base.

Command: `git revert --no-commit edb6dec c5582fd 26076b0 && git commit -m "revert: remove Vercel proxy attempt, restore clean Railway-only setup"`

## DNS (manual — user action required)

At domain registrar for `deyvyd.com`:

```
CNAME  ferramentas         →  cname.vercel-dns.com
CNAME  docgen.ferramentas  →  [value shown in Railway dashboard > Settings > Domains]
```

## Railway (manual — user action required)

In Railway dashboard → project → Settings → Domains:
- Add custom domain: `docgen.ferramentas.deyvyd.com`
- Copy the CNAME target value for DNS above

## Vercel (manual — user action required)

In Vercel dashboard:
- Rename project from `portal-trabalho` to `portal-ferramentas` (or match new repo name)
- Remove old domain `trabalho.deyvyd.com`
- Add new domain `ferramentas.deyvyd.com`

## GitHub (manual — user action required)

Rename repo `portal-trabalho` → `portal-ferramentas` in GitHub Settings.
GitHub auto-redirects old URLs so existing clones keep working.
