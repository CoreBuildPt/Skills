# CRM Builder & Local Business Website Generator

Build and maintain CRM platforms that automate finding local businesses, generating professional websites, and managing outreach campaigns.

## Core Architecture

### Stack
- Next.js 16+ (App Router, Server Components, params are Promises)
- TypeScript, Tailwind CSS v4
- Dark theme CRM (zinc-950 bg, emerald-600 primary)
- JSON flat-file persistence (data/ directory)
- Resend (email), Nodemailer (SMTP)
- OpenStreetMap (scraping), Unsplash (images)

### Design Principles
- Dark glassmorphism UI with emerald accent
- Mobile-first responsive (320px to 1920px)
- WCAG AA contrast (4.5:1 minimum)
- PT-PT with correct accents
- No em-dashes in user-visible text (use periods, commas, or pipes)
- No emojis in generated websites (use SVG icons)

### Generated Websites Must Have
1. Hero section with category-appropriate Unsplash image (16:9, crop=center)
2. Business name as wordmark in nav
3. Services section with SVG icon cards (glassmorphism)
4. About section with secondary image
5. Contact section with phone, email, address, Google Maps embed
6. Opening hours (if available)
7. WhatsApp floating button (bottom-right)
8. Cookie banner RGPD compliant (equal-weight Accept/Reject buttons)
9. Privacy Policy as modal (opened from footer link)
10. Livro de Reclamacoes link in footer (www.livroreclamacoes.pt)
11. Company identification in footer (name, NIF, address)
12. Copyright with dynamic year
13. Scroll reveal animations (IntersectionObserver)

### Legal Requirements (Portugal)
- RGPD compliance (Regulamento UE 2016/679)
- Cookie consent with genuine choice (Lei 41/2004)
- Livro de Reclamacoes Eletronico link (obrigatorio)
- Company identification (nome, NIF, morada)
- Allergen info for food businesses (Reg. UE 1169/2011)
- Privacy policy accessible from footer

### Image Selection Rules
- Analyze business NAME before category for image context
- "Barbearia" = male-only images (barbershop interior, tools, chairs)
- "Salao" / "Cabeleireira" = female-focused images
- "Cabeleireiro" = neutral/unisex
- Download and store images locally (data/images/) so they survive source deletion
- Use 1600x900 for hero, 800x500 for sections, always crop=center
- Verify URLs return 200 before using

### Color System
- 18 industry-specific palettes (always white bg for light, #0a0a0a for dark)
- adaptPaletteForTheme() ensures WCAG contrast automatically
- 6 palette options: Original, Warm, Complementary, Triadic, Emerald, Neutral
- Logo fetching: Clearbit > Google Favicon > Facebook > industry default

### Security Checklist
- Crypto tokens (crypto.randomBytes 32 bytes)
- No hardcoded passwords (fail if env not set)
- Timing-safe password comparison
- Rate limiting on login (5 attempts / 15 min)
- Path traversal validation on file routes
- Iframe sandbox without allow-same-origin
- Security headers (X-Frame-Options, X-Content-Type-Options, etc)
- SameSite=strict cookies, 7-day maxAge

### CRM Features
- Dashboard with 8 KPI widgets
- Kanban board (drag-and-drop state changes)
- Pipeline with OSM scraping (Overpass + Nominatim)
- Funnel visualization with percentages
- Automation suggestions (follow-up, email, site generation)
- Activity feed (recent contact events)
- Site version history (6 versions per lead, restore any)
- 3 templates: Modern (Syne+Inter), Classic (Playfair+Inter), Minimal (Space Mono+Inter)
- Dark/Light theme toggle for generated sites
- 6 color palette options per client
- Payment tracking, invoicing, collection reminders
- Editable pricing with margin calculator

### Common Pitfalls
- Overpass queries: use `out center N;` (NOT `out center tags;`)
- Dense cities (Lisboa, Porto): max 5km radius to avoid timeout
- overflow-x:hidden breaks sticky nav (use overflow-x:clip)
- Next.js 16: params are Promises, must await
- Hero images: avoid close-up faces, use wide-angle interiors
- Always kill port before restarting dev server
- Never delete .next cache
