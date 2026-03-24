# OpenClaw for Entrepreneurs — Landing Page

**Site:** openclawforentrepreneurs.com
**Stack:** Static HTML + CSS + vanilla JS — no build step, no framework
**Hosting:** GitHub Pages (recommended) or any static host

---

## Files

```
website/
├── index.html      ← Landing page (all sections, vanilla JS accordion + form)
├── styles.css      ← Extracted stylesheet (dark theme, violet accent)
└── README.md       ← This file
```

---

## Before Going Live — Checklist

Replace these placeholders before pointing traffic at the site:

| Placeholder | Location | What to put there |
|---|---|---|
| `YOUR_FORMSPREE_ID` | `index.html` → `<form action>` | Your Formspree form ID (e.g. `xyzabcde`) |
| `PLACEHOLDER_STRIPE_LINK` | `index.html` → final CTA button | Stripe payment link (e.g. `https://buy.stripe.com/abc123`) |
| `DISCORD_INVITE_LINK_PLACEHOLDER` | `index.html` → footer | Your Discord invite URL |
| `og-image.png` | Root of repo | 1200×630 dark-theme hero screenshot for social sharing |

### Creating the Formspree endpoint
1. Go to [formspree.io](https://formspree.io) → sign up free
2. Create a new form → copy the form ID (the part after `/f/`)
3. Replace `YOUR_FORMSPREE_ID` in `index.html` with your ID
4. **Test it:** submit a real email and confirm it lands in your Formspree dashboard

> **Note:** Until the Formspree ID is replaced, the form falls back to saving
> emails in `localStorage` — no lead is lost during development.

### Creating the Stripe payment link
1. Stripe Dashboard → **Payment Links** → **Create link**
2. Add product: "Marcus AI OS — Early Adopter", $20/month recurring
3. Copy the generated URL (format: `https://buy.stripe.com/[id]`)
4. Replace `PLACEHOLDER_STRIPE_LINK` in `index.html`

---

## Deployment — GitHub Pages

### First time setup

```bash
# 1. Create a new GitHub repo (e.g. openclawforentrepreneurs-site)
git init
git remote add origin git@github.com:YOUR_USERNAME/openclawforentrepreneurs-site.git

# 2. Add the website files
git add website/
git commit -m "BUILD: Landing page — openclawforentrepreneurs.com"

# 3. Push to GitHub
git push -u origin main

# 4. Enable GitHub Pages
# GitHub repo → Settings → Pages → Source: Deploy from branch
# Branch: main, Folder: /website  (or move files to root)
# Save → wait ~60 seconds → site is live at YOUR_USERNAME.github.io/repo-name
```

### Custom domain setup

```bash
# In your DNS provider, add a CNAME record:
# Name:  openclawforentrepreneurs.com
# Value: YOUR_GITHUB_USERNAME.github.io

# In GitHub repo → Settings → Pages → Custom domain
# Enter: openclawforentrepreneurs.com → Save
# GitHub auto-provisions a Let's Encrypt SSL cert within ~10 minutes
```

### Updating the site

```bash
# Edit files locally, then:
git add website/
git commit -m "UPDATE: [what you changed]"
git push
# GitHub Pages re-deploys automatically within ~60 seconds
```

---

## Deployment — Netlify (alternative)

```bash
# Option A: Drag-and-drop the /website folder into netlify.com/drop
# Option B: Connect GitHub repo → set publish directory to "website"
# Netlify handles SSL, CDN, and form handling natively
```

---

## Analytics (add after launch)

The spec calls for **no Google Analytics**. Recommended alternatives:

- **Plausible** ($9/mo) — cookie-free, GDPR-clean, lightweight script
- **Fathom** ($15/mo) — similar, privacy-first
- **Umami** (free, self-hosted) — open-source, runs on Vercel

Add the analytics snippet just before `</body>` in `index.html`.

---

## What to Build Next

Priority order from `WEBSITE-SPEC.md`:

1. **`thank-you.html`** — shown after Stripe checkout (confirmation + Discord link + next steps)
2. **`privacy.html` / `terms.html`** — required before running paid ads (use Termly generator)
3. **Email capture A/B test** — free waitlist vs. $20/mo direct, test at 500 visitors
4. **`/demo.html`** — long-form walkthrough page for high-intent visitors
5. **`docs.openclawforentrepreneurs.com`** — Docusaurus or Markdown on Pages subdomain

---

## Design System Quick Reference

| Token | Value |
|---|---|
| Background | `#0a0a0a` |
| Surface | `#111111` |
| Border | `#1e1e1e` |
| Text primary | `#f0f0f0` |
| Text muted | `#888888` |
| Accent (violet) | `#7c3aed` |
| Accent light | `#9d5cf6` |
| Font | Inter (Google Fonts) |
| Mono font | JetBrains Mono |
| Max content width | 800px |
| Section padding | 96px vertical |
| Border radius | 8px (cards), 4px (buttons) |

---

## Contact

**Email:** hello@openclawforentrepreneurs.com
**Site:** openclawforentrepreneurs.com
**Discord:** [link in footer once set]
