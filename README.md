# Lumiaura Website

**Live:** https://lumiaura.ca  
**Platform:** Cloudflare Pages (Upload assets)  
**Project name:** lumiaura-site-rev1  
**Preview URL:** https://lumiaura-site-rev1.pages.dev  
**GitHub:** https://github.com/mbtheme/LumiAura

## Stack

Site statique pur — HTML + CSS + JS. Aucun framework, aucun build step.

## Structure

```
lumiaura-site/
├── index.html          # Main page (all sections)
├── assets/
│   ├── css/
│   │   └── style.css   # All styles
│   ├── images/         # Photos + logo
│   └── videos/         # Hero + IG reel videos
```

## Run Locally

```bash
# Option 1: any static server
npx serve -s .

# Option 2: Python
python3 -m http.server 8080
```

## Deploy

Deploy via Cloudflare Pages with wrangler:

```bash
CLOUDFLARE_API_TOKEN=$(cat ~/.config/cloudflare/env | grep TOKEN | cut -d= -f2) \
  wrangler pages deploy /home/geppo/.openclaw/workspace/lumiaura-site/ \
  --project-name=lumiaura-site-rev1 --branch=production
```

## Form Submissions

Quote form sends to `lumiaura1@outlook.com` via formsubmit.co.

## Domain

- Registered via Shopify (Tucows/OpenSRS)
- Nameservers: Cloudflare (lucy.ns.cloudflare.com, trevor.ns.cloudflare.com)
- CNAME: lumiaura.ca → lumiaura-site-rev1.pages.dev

## Content Sources

- Instagram: @lumiaura_inc / @lumiaura.ai2026 (via Zernio API)
- Google Reviews: https://maps.app.goo.gl/21ME8Xup1hRZbeRLA
- Phone: 514-923-6906 (Matteo)
- Email: lumiaura1@outlook.com
