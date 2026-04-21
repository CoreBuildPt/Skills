# CoreBuild Skills

Coleção de skills para Claude Code / AI coding agents, curada pela CoreBuild.

---

## 📦 Conteúdo

### `frontend-design/` — by [Anthropic official](https://github.com/anthropics/skills)
Skill oficial da Anthropic para design de frontend. Guia Claude Code em decisões de design visual, hierarchy, spacing, color theory e padrões modernos.

### `ui-skills/` — by [ibelick](https://github.com/ibelick/ui-skills)
Pequenas skills focadas em fundamentos UI/UX:
- **baseline-ui** — design system fundamentals
- **fixing-accessibility** — a11y audit/fixes
- **fixing-metadata** — meta tags / SEO
- **fixing-motion-performance** — animation perf

### `ui-ux-pro-max-skill/` — by [nextlevelbuilder](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)
Antigravity Kit: toolkit AI-powered de design intelligence.
Search databases para UI styles, color palettes, font pairings, chart types, UX guidelines.

**Uso principal:**
```bash
python3 src/ui-ux-pro-max/scripts/search.py "<query>" --domain style
```

**Domínios:** product · style · typography · color · landing · chart · ux
**Stacks:** html-tailwind · react · nextjs · astro · vue · nuxtjs · svelte · swiftui · react-native · flutter · shadcn · jetpack-compose

### `marketing-growth-skills/` — 50 skills Marketing · CRO · Dev workflow
Coleção completa de skills de marketing, growth, copywriting e workflow de desenvolvimento.

**Categorias principais:**

**🎯 CRO & Conversion** (6 skills)
- `form-cro` · `onboarding-cro` · `page-cro` · `paywall-upgrade-cro` · `popup-cro` · `signup-flow-cro`

**✍️ Copywriting & Content** (7 skills)
- `copywriting` · `copy-editing` · `content-strategy` · `writing-plans` · `writing-skills` · `social-content` · `email-sequence`

**📊 SEO & Discovery** (5 skills)
- `ai-seo` · `seo-audit` · `programmatic-seo` · `schema-markup` · `site-architecture`

**💰 Pricing & Strategy** (5 skills)
- `pricing-strategy` · `launch-strategy` · `free-tool-strategy` · `competitor-alternatives` · `product-marketing-context`

**🎨 Advertising & Creative** (3 skills)
- `ad-creative` · `paid-ads` · `ab-test-setup`

**📧 Lead Gen & Email** (4 skills)
- `cold-email` · `email-sequence` · `lead-magnets` · `referral-program`

**🧠 Psychology & Research** (4 skills)
- `marketing-psychology` · `customer-research` · `brainstorming` · `marketing-ideas`

**🔁 Retention & Growth** (3 skills)
- `churn-prevention` · `community-marketing` · `revops`

**📱 Mobile & App** (2 skills)
- `aso-audit` · `analytics-tracking`

**🛠️ Dev Workflow** (10 skills)
- `subagent-driven-development` · `dispatching-parallel-agents` · `executing-plans` · `writing-plans`
- `test-driven-development` · `systematic-debugging` · `verification-before-completion`
- `requesting-code-review` · `receiving-code-review` · `finishing-a-development-branch`
- `using-git-worktrees` · `using-superpowers`

**📞 Sales** (1 skill)
- `sales-enablement`

---

## 🚀 Como usar num projeto

1. Clonar este repo no home folder:
   ```bash
   git clone https://github.com/CoreBuildPt/Skills ~/.claude/skills-corebuild
   ```

2. Claude Code detecta skills automaticamente em `~/.claude/skills/`

3. Copiar pastas/ficheiros relevantes conforme o projeto:
   ```bash
   # UI-focused projects
   cp -r ~/.claude/skills-corebuild/ui-ux-pro-max-skill ~/.claude/skills/

   # Marketing/growth projects
   cp ~/.claude/skills-corebuild/marketing-growth-skills/copywriting-SKILL.md ~/.claude/skills/
   ```

---

## 🗺️ Roadmap

- [ ] Skill custom de design references (baseada em impeccable.style)
- [ ] Skill Portuguese/Spanish SEO (BLABLABOAT specific)
- [ ] Skill Next.js + Prisma + Postgres boilerplate
- [ ] Skill charter/tourism booking UX (BLABLABOAT learnings)

---

## 📜 Licença
Cada sub-skill mantém a sua licença original. Ver LICENSE files respetivos dentro de cada pasta.
