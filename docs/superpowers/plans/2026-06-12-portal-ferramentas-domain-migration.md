# Portal Ferramentas — Domain Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rename portal from `trabalho.deyvyd.com` to `ferramentas.deyvyd.com`, point docgen card to `https://docgen.ferramentas.deyvyd.com`, and revert gerador-de-documentacoes to its clean pre-proxy state.

**Architecture:** Two repos, two deployments. `portal-ferramentas` (static HTML) → Vercel at `ferramentas.deyvyd.com`. `gerador-de-documentacoes` (Flask+Vue) → Railway at `docgen.ferramentas.deyvyd.com`. No proxy between them — direct Railway subdomain.

**Tech Stack:** Static HTML/CSS, Vercel, Railway, git revert.

---

## File Map

### Repo: portal-trabalho (local path: `C:\Users\Deyvy\Downloads\portal-trabalho`)

| File | Change |
|------|--------|
| `vercel.json` | Remove all Railway rewrites; keep only root redirect |
| `aiyra/index.html` | Nav brand, title, docgen card href |
| `aiyra/asap-chamados/index.html:306` | Nav brand `trabalho` → `ferramentas` |
| `aiyra/asap-devsecops/index.html:283` | Nav brand `trabalho` → `ferramentas` |
| `aiyra/gerador-relatorio-sustentacao/index.html:438` | Nav brand `trabalho` → `ferramentas` |

### Repo: gerador-de-documentacoes (local path: `C:\Users\Deyvy\Downloads\gerador-de-documentacoes`)

| Action | Detail |
|--------|--------|
| `git revert` | Revert commits `26076b0`, `c5582fd`, `edb6dec` (Vercel proxy attempt) |

---

## Task 1: Update vercel.json — remove Railway rewrites

**Repo:** `portal-trabalho`

**Files:**
- Modify: `vercel.json`

- [ ] **Step 1: Replace vercel.json content**

Replace entire file with:

```json
{
  "redirects": [
    { "source": "/", "destination": "/aiyra/", "permanent": true }
  ]
}
```

- [ ] **Step 2: Verify file**

Open `vercel.json` and confirm it has only the redirects block — no `rewrites` key at all.

- [ ] **Step 3: Commit**

```bash
git add vercel.json
git commit -m "fix(vercel): remove Railway proxy rewrites, portal is static-only"
```

---

## Task 2: Update portal landing page (aiyra/index.html)

**Repo:** `portal-trabalho`

**Files:**
- Modify: `aiyra/index.html`

Three changes in this file:

1. **Line 6** — `<title>`: `Aiyra — Ferramentas de trabalho` → `Aiyra — Ferramentas · Deyvyd Moura`
2. **Line 17** — nav brand: `trabalho` → `ferramentas`
3. **Line 30** — docgen card href: `/aiyra/docgen/` → `https://docgen.ferramentas.deyvyd.com`

- [ ] **Step 1: Update title (line 6)**

```html
<title>Aiyra — Ferramentas · Deyvyd Moura</title>
```

- [ ] **Step 2: Update nav brand (line 17)**

```html
ferramentas<span class="nav-dot"></span><span>deyvyd moura</span>
```

- [ ] **Step 3: Update docgen card href (line 30)**

```html
<a href="https://docgen.ferramentas.deyvyd.com" class="featured">
```

- [ ] **Step 4: Verify in browser**

Open `aiyra/index.html` locally (double-click or `start aiyra/index.html`). Confirm:
- Browser tab shows `Aiyra — Ferramentas · Deyvyd Moura`
- Nav shows `ferramentas · deyvyd moura`
- Docgen card href points to `https://docgen.ferramentas.deyvyd.com`

- [ ] **Step 5: Commit**

```bash
git add aiyra/index.html
git commit -m "feat: rename portal brand trabalho→ferramentas, point docgen to subdomain"
```

---

## Task 3: Update nav brand in tool pages

**Repo:** `portal-trabalho`

**Files:**
- Modify: `aiyra/asap-chamados/index.html` (line 306)
- Modify: `aiyra/asap-devsecops/index.html` (line 283)
- Modify: `aiyra/gerador-relatorio-sustentacao/index.html` (line 438)

Each file has exactly one occurrence of `trabalho` in the nav brand. Replace `trabalho` with `ferramentas` in each.

- [ ] **Step 1: Update asap-chamados nav (line 306)**

Find:
```html
trabalho<span class="portal-nav-dot"></span><span class="portal-brand-accent">deyvyd moura</span>
```

Replace with:
```html
ferramentas<span class="portal-nav-dot"></span><span class="portal-brand-accent">deyvyd moura</span>
```

- [ ] **Step 2: Update asap-devsecops nav (line 283)**

Find:
```html
trabalho<span class="portal-nav-dot"></span><span class="portal-brand-accent">deyvyd moura</span>
```

Replace with:
```html
ferramentas<span class="portal-nav-dot"></span><span class="portal-brand-accent">deyvyd moura</span>
```

- [ ] **Step 3: Update gerador nav (line 438)**

Find (inside Vue template):
```html
trabalho<span class="portal-nav-dot"></span>
```

Replace with:
```html
ferramentas<span class="portal-nav-dot"></span>
```

- [ ] **Step 4: Verify**

Run in PowerShell from repo root:
```powershell
Select-String -Path "aiyra\**\index.html" -Pattern "\btrabalho\b" -Recurse
```
Expected: only the two intentional occurrences in `aiyra/index.html` line 26 (`aceleram o trabalho`) — no nav brand hits.

- [ ] **Step 5: Commit**

```bash
git add aiyra/asap-chamados/index.html aiyra/asap-devsecops/index.html "aiyra/gerador-relatorio-sustentacao/index.html"
git commit -m "feat: update nav brand trabalho→ferramentas in all tool pages"
```

---

## Task 4: Push portal-ferramentas to GitHub

**Repo:** `portal-trabalho`

- [ ] **Step 1: Push all commits**

```bash
git push
```

Expected output ends with `main -> main`.

- [ ] **Step 2: Confirm on GitHub**

Go to `https://github.com/deyvyd/portal-trabalho` (or whatever the remote is). Confirm latest commit is the nav brand update from Task 3.

---

## Task 5: Revert gerador-de-documentacoes to pre-proxy state

**Repo:** `gerador-de-documentacoes` (local path: `C:\Users\Deyvy\Downloads\gerador-de-documentacoes`)

Revert the 3 commits added during the failed Vercel proxy attempt. These introduced Flask-CORS, VITE_API_URL fetch calls, and Vite/Vue Router base paths — none of which are needed when Railway serves the app directly.

Commits to revert (newest first):
- `edb6dec` — fix: rewrite requirements.txt as UTF-8, add Flask-Cors
- `c5582fd` — fix: add Flask-CORS, route API calls via VITE_API_URL env var
- `26076b0` — fix: set base path /aiyra/docgen/ for Vercel proxy

- [ ] **Step 1: Open terminal in gerador-de-documentacoes**

```powershell
cd "C:\Users\Deyvy\Downloads\gerador-de-documentacoes"
```

- [ ] **Step 2: Revert the 3 commits**

```bash
git revert --no-commit edb6dec c5582fd 26076b0
```

Expected: no output (staged changes, not yet committed).

- [ ] **Step 3: Verify staged diff matches expectations**

```bash
git diff --cached --stat
```

Expected files changed:
- `app.py` (CORS lines removed)
- `requirements.txt` (Flask-Cors line removed)
- `src/router/index.js` (base path removed)
- `src/views/DocumentacaoTecnica.vue` (VITE_API_URL fetch calls reverted)
- `src/views/DocumentacaoDesenvolvimento.vue` (VITE_API_URL fetch calls reverted)
- `vite.config.js` (base removed)

- [ ] **Step 4: Spot-check key files**

Check `app.py` — should NOT contain `flask_cors` or `CORS(`:
```bash
git diff --cached app.py
```

Check `vite.config.js` — should NOT contain `base:`:
```bash
git diff --cached vite.config.js
```

- [ ] **Step 5: Commit the revert**

```bash
git commit -m "revert: remove Vercel proxy attempt, restore clean Railway-only setup"
```

- [ ] **Step 6: Push**

```bash
git push
```

Expected: Railway auto-triggers a new deploy. Watch in Railway dashboard → Deployments.

- [ ] **Step 7: Confirm Railway deploy succeeds**

In Railway dashboard, wait for the new deploy to show green. Click "View logs" and confirm no pip errors and the app starts with `Running on http://0.0.0.0:...`.

---

## Task 6: Manual DNS + Railway + Vercel config (user action)

These steps cannot be automated — they require dashboard access.

- [ ] **Step 1: Railway — add custom domain**

1. Go to Railway dashboard → `gerador-de-documentacoes` project
2. Settings → Domains → Add Domain
3. Enter: `docgen.ferramentas.deyvyd.com`
4. Copy the CNAME target value shown (looks like `<random>.up.railway.app`)

- [ ] **Step 2: DNS — add CNAME records**

At your domain registrar for `deyvyd.com`, add two records:

| Type | Name | Value |
|------|------|-------|
| CNAME | `ferramentas` | `cname.vercel-dns.com` |
| CNAME | `docgen.ferramentas` | `<value copied from Railway in Step 1>` |

- [ ] **Step 3: Vercel — update project domain**

1. Go to Vercel dashboard → `portal-trabalho` project → Settings → Domains
2. Remove `trabalho.deyvyd.com`
3. Add `ferramentas.deyvyd.com`
4. Vercel will verify the CNAME automatically once DNS propagates

- [ ] **Step 4: Verify propagation**

DNS propagation can take 1–30 minutes. Test with:
```bash
nslookup ferramentas.deyvyd.com
nslookup docgen.ferramentas.deyvyd.com
```

Both should resolve (no `NXDOMAIN`).

- [ ] **Step 5: Smoke test**

Visit in browser:
- `https://ferramentas.deyvyd.com/aiyra/` — portal landing loads, nav shows `ferramentas · deyvyd moura`
- `https://docgen.ferramentas.deyvyd.com` — docgen Vue app loads
- Click docgen card on portal — opens `https://docgen.ferramentas.deyvyd.com`
- In docgen, generate a document — confirm POST `/gerar_documentos` returns a file (no 405)

---

## Task 7: (Optional) Rename GitHub repo

- [ ] **Step 1: Rename on GitHub**

Go to `https://github.com/deyvyd/portal-trabalho` → Settings → Repository name → change to `portal-ferramentas` → Rename.

GitHub auto-redirects old URLs so existing clones keep working without a `git remote set-url`.

- [ ] **Step 2: Update local remote (optional, for cleanliness)**

```bash
git remote set-url origin https://github.com/deyvyd/portal-ferramentas.git
```
