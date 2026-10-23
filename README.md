# IPC Church Steven: Bilingual Church Website & CMS

A bilingual (English / Telugu) church website with a custom admin CMS, live at **[ipcchurchsteven.com](https://ipcchurchsteven.com)**.

Built for IPC Church, led by Pastor B Steven. Church staff with no technical background can update the whole site from a browser-based admin dashboard, without touching code.

> **About this repository:** this repo documents the project. The site's source code is kept in a separate deployment repository that Cloudflare publishes from.

---

## Live site

- **Website:** https://ipcchurchsteven.com
- **Backup URL:** https://ipcchurchsteven.stevenburla4.workers.dev

---

## Features

### Public website
- **Bilingual:** English ↔ Telugu toggle on every page
- **Hero slideshow** with slides managed in the CMS
- **Watch Live:** embeds the YouTube live stream when the church is on air, and shows the next broadcast time when it's offline
- **Latest Sermons & Videos:** pulls recent videos from the church's YouTube channel automatically
- **Gallery** with several album layouts (grid, bento, masonry, carousel)
- **Pastors** and **Testimonials** (Youth and Member) with photos
- **Kids Programs** page with a photo gallery and collage style for each program
- **Posts / Updates:** event recaps and announcements, with multiple sections per post
- **Song Lyrics** (worship and Sunday school) with search and audio links
- **Church Timings** widget
- **Prayer Request form:** saves the request to the CMS and sends an email notification to the church admin inbox
- **Contact** section with a Google Maps embed
- **Privacy Policy** and **Terms of Use** pages
- **Responsive** layout for phone, tablet and desktop
- **Live updates:** published changes appear on open pages without a refresh (Supabase Realtime)

### Admin CMS
| Area | What it does |
|---|---|
| **Overview** | Dashboard snapshot, **section visibility** toggles for the homepage, maintenance and recovery tools |
| **Layout Manager** | Drag and drop to reorder homepage sections |
| **Media Library** | Central image store with batch upload and in-browser cropping (Cropper.js) |
| **Hero Slider** | Edit homepage slides |
| **Gallery** | Albums, each with its own layout style |
| **Events** | Add and edit church events |
| **Posts / Updates** | Posts with unlimited sections, each with its own title, text and photos |
| **Pastors** | Profiles with photo upload; confirms each photo saved to cloud storage |
| **Testimonials** | Youth and Member categories |
| **Lyrics** | Bilingual lyrics with audio links |
| **Ministries** and **Kids Programs** | Program details, photo albums and collage style |
| **Text Editor** and **Translation Manager** | Edit visible site text in English and Telugu |
| **YouTube** | Paste a channel URL or handle and the channel ID is resolved automatically |
| **Watch Live** | Stream URL, live/offline switch and next-broadcast details |
| **Support** | Donation details (UPI, bank, external donate link) |
| **Prayer Requests** | Inbox of submitted requests |
| **Site Settings** and **SEO** | Church contact info and social links; page titles, descriptions and keywords |
| **User Roles** | Admin accounts and permission levels |
| **Analytics**, **Activity Log** and **Backups** | Content statistics, a history of admin changes and errors, and full CMS backup and restore |
| **Draft / Publish** | Edits stay in a draft until an admin clicks Publish |
| **Idle auto-logout** | Signs out after 5 minutes of inactivity, with a 60-second warning |

---

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | HTML5, CSS3, JavaScript (ES6+), no frameworks |
| Styling | Hand-written CSS (Grid, Flexbox); Google Fonts: Inter, Playfair Display |
| Bilingual text | Custom translation layer using `data-i18n` attributes |
| Image cropping | Cropper.js (CDN) |
| Database | Supabase (PostgreSQL), content stored as JSON in a `cms_data` table |
| Live updates | Supabase Realtime |
| Media storage | Supabase Storage (`church-media` bucket) |
| Firebase compatibility | `supabase-shim.js` provides a Firebase-style API, so the site moved from Firebase to Supabase without rewriting the app |
| Hosting | Cloudflare Workers (static assets), auto-deployed from GitHub |
| Domain and HTTPS | Custom domain through Cloudflare |
| Email | `contact@ipcchurchsteven.com` via Cloudflare Email Routing |
| Prayer notifications | Formsubmit.co |
| YouTube feed | Reads the channel's videos page through r.jina.ai, with a fallback proxy (YouTube's RSS feed was unreliable) |

---

## Architecture

```
┌─────────────────────┐          ┌─────────────────────┐
│  Public visitor     │          │  Church admin       │
│  ipcchurchsteven.com│          │  /admin/dashboard   │
└──────────┬──────────┘          └──────────┬──────────┘
           │                                │
           ▼                                ▼
   ┌──────────────────────────────────────────┐
   │  Cloudflare Workers (static hosting)     │
   │  HTML, CSS, JS for the site and the CMS  │
   └──────────┬───────────────────────────────┘
              │
              ▼
   ┌──────────────────────────────────────────┐
   │  Supabase                                │
   │  • cms_data table: draft + published     │
   │  • Realtime: pushes changes to pages     │
   │  • Storage: church-media (photos)        │
   └──────────────────────────────────────────┘
```

- **Draft and publish:** the admin edits the `state` row (the draft). Clicking **Publish** copies it to the `public` row, which is what visitors see.
- **Live updates:** pages subscribe to changes on `cms_data`, so published content shows up without a reload.
- **Media:** uploads go to the `church-media` storage bucket and are served by public URL.

---

## Code structure

```
/
├── index.html            Homepage
├── kids-programs.html    Kids ministry page
├── testimonials.html     Testimonials page
├── lyrics.html           Song lyrics page
├── privacy.html          Privacy policy
├── terms.html            Terms of use
├── main.js               Public site logic (YouTube feed, prayer form, slideshow)
├── cms-bridge.js         Reads published content and renders it on the page
├── translations.js       English/Telugu text and the language toggle
├── lyrics.js             Lyrics search and display
├── supabase-shim.js      Firebase-style API backed by Supabase
├── style.css             Site styles
├── lyrics.css            Lyrics page and shared footer styles
├── deploy.ps1            PowerShell script: commit and push, which triggers deployment
│
└── admin/                Content management system
    ├── login.html        Sign-in page
    ├── dashboard.html    Dashboard
    ├── admin-core.js     State, sign-in, publish and sync, idle logout
    ├── admin-content.js  Content editors (pastors, gallery, posts, kids…)
    ├── admin-settings.js YouTube, SEO, user roles, analytics, backups, site info
    ├── admin-style.css   Dashboard styles
    └── translations.html Translation Manager
```

---

## How it's deployed

1. Changes are committed and pushed to GitHub with `deploy.ps1`, a small PowerShell script that runs `git add`, `git commit` and `git push` and skips the push when nothing has changed.
2. Cloudflare picks up the push and redeploys the site, usually within a minute.
3. No build step is needed, because the site is plain HTML, CSS and JavaScript.

---

## What I'd improve next

- **Move sign-in to Supabase Auth** and protect the database with Row Level Security policies, so only signed-in admins can change content.
- **Automated tests** for the publish flow and the translation layer.
- **A staging site** to preview changes before they go live.

---

## Credits

Built by Praveen Chandu Pandu for IPC Church, Suryanarayana Colony, Andhra Pradesh, India, using AI-assisted development (Claude, Google Antigravity).

- Content and ministry direction: Pastor B Steven
- Fonts: Google Fonts (Inter, Playfair Display)

## License

Private project. All rights reserved. Content, branding and photos belong to IPC Church.
